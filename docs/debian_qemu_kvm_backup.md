# Backup de Máquinas Virtuais em QEMU+KVM

## O que é Backup de VM

**Backup de máquina virtual** é a replicação sistemática dos arquivos de disco (imagens QCOW2, RAW, VDI, etc.) e metadados de configuração (arquivos XML do libvirt) para um local independente, garantindo recuperação em caso de corrupção, falha de hardware, exclusão acidental ou desastre. Diferencia-se de snapshots, que são pontos de restauração locais; backups são cópias isoladas em mídia ou storage separado.

No contexto de QEMU+KVM, o backup preserva o estado completo da máquina virtual, incluindo sistema operacional, dados, configurações e histórico de inicialização, permitindo restauração total em caso de perda ou dano do repositório primário.

---

## Por Que Fazer Backup de VMs

1. **Proteção contra falhas de hardware**
   
   1.1 Discos podem falhar subitamente; backups isolam o risco
   
   1.2 Storage compartilhado (TrueNAS, NFS) também está sujeito a corrupção de filesystem

2. **Recuperação de desastres**
   
   2.1 Ransomware, malware ou ataques comprometem a máquina e seu storage
   
   2.2 Backup offline ou com retenção temporal oferece defesa em profundidade

3. **Compliance e auditoria**
   
   3.1 Muitos ambientes corporativos exigem histórico de backups documentado
   
   3.2 Demonstra conformidade com políticas de continuidade de negócios

4. **Reversão de mudanças catastróficas**
   
   4.1 Aplicações mal-configuradas ou atualizações problemáticas corrompem o SO
   
   4.2 Backup anterior restaura a máquina para estado operacional

5. **Mobilidade e migração**
   
   5.1 Backups permitem replicar VMs para outro host, datacenter ou arquivar
   
   5.2 Facilita testes, clonagem e disaster recovery

---

## Como Fazer Backup: Estratégias

### 1. Backup a Frio (Máquina Desligada)

1.1 **Vantagens**
   
   a) Garante consistência de dados (sem I/O em progresso)
   
   b) Simples de implementar com `cp` ou `rsync`
   
   c) Ideal para ambientes de desenvolvimento/teste

1.2 **Desvantagens**
   
   a) Downtime da máquina durante a cópia
   
   b) Ineficiente para VMs com discos grandes (demora mais)

1.3 **Quando usar**
   
   a) VMs não-críticas ou em horários de baixa utilização
   
   b) Laboratórios de teste
   
   c) Máquinas que podem ficar offline

### 2. Backup a Quente (Máquina Ligada)

2.1 **Vantagens**
   
   a) Zero downtime; máquina continua operacional
   
   b) Ideal para produção e ambientes críticos

2.2 **Desvantagens**
   
   a) Requer snapshots internos ou ferramentas como `virsh blockcommit`
   
   b) Mais complexo de implementar corretamente
   
   c) Pode impactar performance durante a cópia

2.3 **Quando usar**
   
   a) VMs em produção
   
   b) Servidores que não podem parar
   
   c) Ambientes com SLA crítico

### 3. Backup Incremental

3.1 **Conceito**
   
   a) Primeira execução: cópia completa (full backup)
   
   b) Execuções subsequentes: apenas blocos alterados (delta)
   
   c) Reduz tempo e espaço em storage

3.2 **Implementação**
   
   a) QEMU suporta via `qemu-img convert` com snapshots
   
   b) Ferramentas como `virt-manager` abstraem essa complexidade

---

## Implementação Prática: Backup a Frio com Cópia Direta

### Passo 1: Desligar a Máquina Virtual

```bash
virsh shutdown win2k25
```

Aguarde a máquina desligar completamente (verificar com `virsh list --all`).

### Passo 2: Copiar a Imagem QCOW2 para o Destino

```bash
cp ~/libvirt/images/win2k25.qcow2 /media/backup-vm/win2k25.qcow2.backup-$(date +%Y%m%d-%H%M%S)
```

**Explicação:**
- Copia o arquivo de origem para destino com timestamp
- Exemplo de nome gerado: `win2k25.qcow2.backup-20250206-143022`
- Timestamp evita sobrescrita de backups anteriores

### Passo 3: Verificar Integridade do Backup

```bash
qemu-img info /media/backup-vm/win2k25.qcow2.backup-*
```

Saída esperada mostra tamanho virtual, tamanho real em disco (formato QCOW2) e checksum.

### Passo 4: Reiniciar a Máquina Virtual

```bash
virsh start win2k25
```

---

## Backup Automatizado com Script e Montagem de Disco

O script abaixo automatiza backup a frio, montando o disco identificado pelo label `backup-vms`, assim, os discos externos que usar para este fim terão que ter este label ou você - eu mostro mais embaixo - especifica como parametro o label que indentificará o disco que irá usar. Este script não funcionará com discos que não tem label, use o `gparted` se tiver discos que precisará nomear.  

Dessa forma, o script será capaz de identificar o disco sozinho, montando-o e executando a cópia em subpasta dedicada por VM, e desmontando em seguida, Então crie o script `backup-vm.sh` de backup com o seguinte conteúdo:

```bash
#!/bin/bash
# backup-vm.sh - Backup automatizado de máquinas virtuais QEMU+KVM
# Autor: Gladiston Santana <gladiston.santana[em]gmail.com>
# Uso: sudo ./backup-vm.sh <caminho-da-vm> <label-ou-caminho-destino>
# Exemplo:
#   sudo ./backup-vm.sh ~/libvirt/images/win2k25.qcow2 "#hist"
#   sudo ./backup-vm.sh ~/libvirt/images/win2k25.qcow2 /mnt/backup
# Licença: MIT (Permissiva)
# Criado em: 06/11/2025
# Ult. Atualização: 06/11/2025
#
# Descrição:
#   Backup de VM QEMU/KVM com destino inteligente (label ou diretório),
#   cópia via rsync preservando sparse, verificação qemu-img e checksum MD5 opcional.
#########################################################

set -euo pipefail
START_TS=$(date +%s)

# --- CONFIGURAÇÕES GERAIS ---
TIMESTAMP_HOUR=false            # se true, adiciona "-HHh" ao nome do backup
SKIP_CHECKSUM=false             # se true, pula geração de checksum MD5
BACKUP_ROOT_NAME="libvirt-bak"  # nome da pasta principal de destino

# --- VALIDAÇÃO DE PARÂMETROS ---
if [ $# -lt 2 ]; then
  echo "ERRO: Parâmetros insuficientes."
  echo "Uso: sudo $0 <caminho-da-vm> <label-ou-caminho-destino>"
  exit 1
fi

VM_PATH_RAW="${1}"
DEST_PARAM="${2}"

# monta timestamp conforme configuração
if [ "${TIMESTAMP_HOUR}" = true ]; then
  TIMESTAMP=$(date +%Y%m%d-%Hh)
else
  TIMESTAMP=$(date +%Y%m%d)
fi

MOUNT_POINT=""
VM_STATE="undefined"
WAS_RUNNING=0

# --- FUNÇÕES ---
log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*"; }

error_exit() { log "--- ERRO FATAL: $1 ---"; exit 1; }

cleanup() {
  log "--- Iniciando rotina de limpeza ---"
  if [ -n "${MOUNT_POINT}" ] && mountpoint -q "${MOUNT_POINT}" 2>/dev/null; then
    log "Desmontando ${MOUNT_POINT}..."
    umount "${MOUNT_POINT}" || log "AVISO: falha ao desmontar ${MOUNT_POINT}"
  fi
  if [ -n "${MOUNT_POINT}" ] && [ -d "${MOUNT_POINT}" ]; then
    rmdir "${MOUNT_POINT}" 2>/dev/null || true
  fi
  log "--- Limpeza concluída ---"
}

fmt_dur() {
  local s="$1"
  local h=$(( s/3600 ))
  local m=$(( (s%3600)/60 ))
  local ss=$(( s%60 ))
  local out=""
  [ $h -gt 0 ] && out="${out}${h}h "
  [ $m -gt 0 ] && out="${out}${m}m "
  out="${out}${ss}s"
  echo "$out"
}

trap cleanup EXIT INT TERM

# ===== 1. PERMISSÕES =====
log "Verificando permissões..."
[ "${EUID}" -ne 0 ] && error_exit "Este script precisa ser executado como root."

# ===== 2. EXPANSÃO DE CAMINHO =====
if [[ "${VM_PATH_RAW}" == "~/"* ]]; then
  SUDO_HOME=$(getent passwd "${SUDO_USER:-root}" | cut -d: -f6)
  VM_PATH="${SUDO_HOME}/${VM_PATH_RAW:2}"
else
  VM_PATH="${VM_PATH_RAW}"
fi
[ ! -f "${VM_PATH}" ] && error_exit "Arquivo da VM não encontrado: ${VM_PATH}"

VM_FILE=$(basename "${VM_PATH}")
VM_NAME="${VM_FILE%.*}"
log "VM: ${VM_NAME} (${VM_PATH})"

# ===== 3. DESTINO =====
if [ -d "${DEST_PARAM}" ]; then
  DEST_DIR="${DEST_PARAM}"
  log "Destino detectado como diretório: ${DEST_DIR}"
else
  log "Procurando disco com label '${DEST_PARAM}'..."
  DEVICE_PATH=$(findfs "LABEL=${DEST_PARAM}" 2>/dev/null || true)
  [ -z "${DEVICE_PATH}" ] && error_exit "Destino '${DEST_PARAM}' não é diretório nem label válido."
  MOUNT_POINT="/mnt/backup-temp-${RANDOM}"
  mkdir -p "${MOUNT_POINT}"
  log "Montando ${DEVICE_PATH} em ${MOUNT_POINT}..."
  mount "${DEVICE_PATH}" "${MOUNT_POINT}"
  DEST_DIR="${MOUNT_POINT}"
  log "Disco montado com sucesso em ${MOUNT_POINT}"
fi

# ===== 4. ESTRUTURA DE PASTAS E PERMISSÕES =====
BACKUP_ROOT="${DEST_DIR}/${BACKUP_ROOT_NAME}"
BACKUP_SUBDIR="${BACKUP_ROOT}/${VM_NAME}"
mkdir -p "${BACKUP_SUBDIR}"
chmod -R 777 "${BACKUP_ROOT}"
log "Destino final: ${BACKUP_SUBDIR}"

# ===== 5. GERENCIAMENTO DE VM =====
if command -v virsh >/dev/null 2>&1 && virsh dominfo "${VM_NAME}" >/dev/null 2>&1; then
  VM_STATE=$(virsh domstate "${VM_NAME}" | head -n1)
  log "Estado atual da VM: ${VM_STATE}"
  if echo "${VM_STATE}" | grep -qi running; then
    WAS_RUNNING=1
    log "Solicitando desligamento de '${VM_NAME}'..."
    virsh shutdown "${VM_NAME}" || true
    for i in $(seq 1 60); do
      sleep 1
      state_now=$(virsh domstate "${VM_NAME}" | head -n1 || true)
      if echo "${state_now}" | grep -qi "shut"; then
        log "VM desligada."
        break
      fi
      if [ "${i}" -eq 60 ]; then
        log "Forçando desligamento (destroy)."
        virsh destroy "${VM_NAME}" || true
      fi
    done
  fi
fi

# ===== 6. BACKUP =====
BACKUP_FILE="${BACKUP_SUBDIR}/${VM_FILE}.backup-${TIMESTAMP}"
log "Iniciando cópia com rsync..."
rsync -ah --info=progress2 --inplace --sparse --no-whole-file "${VM_PATH}" "${BACKUP_FILE}"
sync
log "Cópia concluída."

# ===== 7. VERIFICAÇÃO =====
if [[ "${VM_FILE}" == *.qcow2 ]]; then
  if qemu-img check "${BACKUP_FILE}" >/dev/null 2>&1; then
    log "Integridade QCOW2 OK."
  else
    error_exit "Falha na verificação QCOW2."
  fi
fi

# ===== 8. CHECKSUM (opcional) =====
if [ "${SKIP_CHECKSUM}" = false ]; then
  log "Gerando checksum (MD5)..."
  md5sum "${BACKUP_FILE}" > "${BACKUP_FILE}.md5"
  sync
  log "Checksum salvo em: ${BACKUP_FILE}.md5"
else
  log "Checksum ignorado (SKIP_CHECKSUM=true)."
fi

# ===== 9. REINICIAR VM =====
if command -v virsh >/dev/null 2>&1 && [ "${WAS_RUNNING}" -eq 1 ]; then
  log "Reiniciando VM '${VM_NAME}'..."
  virsh start "${VM_NAME}" || log "Falha ao iniciar VM."
fi

# ===== 10. RESULTADO + DURAÇÃO =====
ELAPSED=$(( $(date +%s) - START_TS ))
log "================================================="
log "=== BACKUP CONCLUÍDO COM SUCESSO ==="
log "================================================="
log "Arquivo: ${BACKUP_FILE}"
log "Tamanho: $(du -h "${BACKUP_FILE}" | cut -f1)"
log "Duração total: $(fmt_dur "${ELAPSED}")"

```

Em servidores, voce provavelmente precisará de um esquema de script diferente, discos externos por dia da semana, backup incremental/diferencial, teste de checksum o qual não é nosso proposito aqui em demonstrar. Nosso uso tá voltado para virtualizar em desktop e por isso, o script acima é suficiente.   

O script gera um arquivo final assim:
```
[destino]/libvirt-bak/win2k25/win2k25.qcow2.backup-20251106-14h
```
O script usa `rsync` para fazer transferências, então se você repetir dois backups no mesmo dia, o ultimo substituirá o anterior, mas note que ele fará isso usando o `rsync` que fará o uso para transferir apenas o **delta** do arquivo velho para o novo.  
Caso queira uma granularidade de hora em hora, edite a variavel **TIMESTAMP_HOUR** para **true** no script e então o backup só usará delta no ntervalo da mesma hora, ou seja, o ultimo substituirá o anterior, mas somente no intervalo da mesma hora, por exemplo, entre 14h00 e 14h59 haverá apenas um unico backup não importa o horario que executar e se executar outro às 13h00, este ultimo não substituirá o backup das 14h.  

## Uso do Script
Tornar executável:  
```bash
chmod +x backup-vm.sh
```
Executar backup (sintaxe: sudo ./backup-vm.sh /local/da/vm.qcow2 [label-disco|caminho])  
Se o segundo parametro for omitido, ele procurará como destino um disco que tenha o label **backup-vms**.

Mas dá para usar discos com outros *labels* também, veja:
```bash
$ lsblk -f
NAME        FSTYPE FSVER LABEL   UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                  
└─sda1      ext4   1.0   #hist   943b591a-35de-4079-8b65-4bd81b1e32ed    1,5T    13% /media/hist
sdb                                                                                  
└─sdb1      ext4   1.0   #dados2 b2154643-7b94-42a1-8146-267bb88ba833  565,4G    33% /home/gsantana/work/WinSrv
                                                                                     /mnt/dados2
(...)
```
Agora vamos fazer o backup para o disco com o label **#hist**, executando:
```bash
sudo ./backup-vm.sh ~/libvirt/images/win2k25.qcow2 "#hist"
```
Mas ao inves de usar o label, também podemos especificar o caminho executando:
```bash
sudo ./backup-vm.sh ~/libvirt/images/win2k25.qcow2 /mnt/dados2
```

---

## Estrutura de Diretórios Resultante

Após executar o script, a organização de backups será:

```
/media/backup-vm/
├── win2k25/
│   ├── win2k25.qcow2.backup-20250206
│   ├── win2k25.qcow2.backup-20250206.md5sum
│   ├── win2k25.qcow2.backup-20250207
│   └── win2k25.qcow2.backup-20250207.md5sum
├── ubuntuserver/
│   ├── ubuntuserver.qcow2.backup-20250206
│   └── ubuntuserver.qcow2.backup-20250206.md5sum
└── debian12/
    ├── debian12.qcow2.backup-20250206
    └── debian12.qcow2.backup-20250206.md5sum
```

Cada VM possui sua própria subpasta isolada, facilitando **retenção seletiva**, **políticas de limpeza por VM** e **auditoria granular** de backups corporativos.

---

## Agendamento com Cron

Para backup automático diário às 2h da manhã:

```bash
sudo editor /etc/crontab
```
E adicionar linha:
```
0 2 * * * /caminho/para/backup-vm.sh /local/da/vm/win2k25.qcow2 backup-vms >> /var/log/backup-vm-cron.log 2>&1
```

Para múltiplas VMs em sequência:

```bash
0 2 * * * /caminho/para/backup-vm.sh /local/da/vm/win2k25.qcow2 backup-vms >> /var/log/backup-vm-cron.log 2>&1
30 2 * * * /caminho/para/backup-vm.sh /local/da/vm/ubuntuserver.qcow2 backup-vms >> /var/log/backup-vm-cron.log 2>&1
0 3 * * * /caminho/para/backup-vm.sh /local/da/vm/debian12.qcow2 backup-vms >> /var/log/backup-vm-cron.log 2>&1
```

No exemplo acima, você deve ter certeza de que os intervalos de backup para o outro são suficientes, porque o script ejeta o disco no final e não poderia fazer isso se outro backup tiver sido iniciado.  

---

## Verificação e Restauração de Backup

### Listar backups disponíveis

```bash
ls -lh /media/backup-vm/libvirt-bak/win2k25/
```

Saída esperada:

```
-rw-r--r-- 1 root root 50G Feb  6 14:30 win2k25.qcow2.backup-20250206
-rw-r--r-- 1 root root 128 Feb  6 14:31 win2k25.qcow2.backup-20250206.md5sum
-rw-r--r-- 1 root root 50G Feb  7 02:00 win2k25.qcow2.backup-20250207
-rw-r--r-- 1 root root 128 Feb  7 02:01 win2k25.qcow2.backup-20250207.md5sum
```

### Validar checksum antes de restaurar

```bash
md5sum -c /media/backup-vm/libvirt-bak/win2k25/win2k25.qcow2.backup-20250206-143022.md5sum
```

Saída esperada:

```
win2k25.qcow2.backup-20250206-143022: OK
```

### Restaurar a partir de backup

```bash
# 1. Parar a VM em produção
virsh destroy win2k25

# 2. Fazer backup da versão atual comprometida (opcional, recomendado)
mv ~/libvirt/images/win2k25.qcow2 ~/libvirt/images/win2k25.qcow2.corrupted-$(date +%Y%m%d)

# 3. Copiar backup para origem
cp /media/backup-vm/libvirt-bak/win2k25/win2k25.qcow2.backup-20250206-143022 ~/libvirt/images/win2k25.qcow2

# 4. Verificar integridade
qemu-img check ~/libvirt/images/win2k25.qcow2

# 5. Reiniciar VM
virsh start win2k25
```

---

## Manutenção e Limpeza de Backups Antigos

### Listar backups por antigüidade

```bash
ls -lht /media/backup-vm/libvirt-bak/win2k25/ | head -10
```

### Remover backups com mais de 30 dias

```bash
find /media/backup-vm/libvirt-bak/win2k25/ -name "*.backup-*" -mtime +30 -delete
```

### Calcular espaço utilizado por VM

```bash
du -sh /media/backup-vm/libvirt-bak/*/
```

---

## Monitoramento e Alertas

### Script de verificação de saúde de backups

```bash
#!/bin/bash

# Verificar integridade de todos os backups
for backup in /media/backup-vm/libvirt-bak/*/*.qcow2.backup-*; do
    [ -f "${backup}" ] || continue
    
    echo "Verificando: ${backup}"
    
    if qemu-img check "${backup}" > /dev/null 2>&1; then
        echo "✓ OK"
    else
        echo "✗ ERRO: Backup corrompido"
    fi
done
```

Executar semanalmente via cron:

```bash
0 3 * * 0 /caminho/para/verify-backups.sh | mail -s "Verificação de Backups" admin@empresa.com
```

---

[Retornar à página de Virtualização nativa com QAEMU+KVM Usando VM/Windows](debian_qemu_kvm_windows.md)   

[Retornar à página de Virtualização nativa com QAEMU+KVM](debian_qemu_kvm.md)   



