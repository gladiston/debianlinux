Vou regenerar a seção completa com o script atualizado:

---

## Backup de Máquinas Virtuais em QEMU+KVM

### O que é Backup de VM

**Backup de máquina virtual** é a replicação sistemática dos arquivos de disco (imagens QCOW2, RAW, VDI, etc.) e metadados de configuração (arquivos XML do libvirt) para um local independente, garantindo recuperação em caso de corrupção, falha de hardware, exclusão acidental ou desastre. Diferencia-se de snapshots, que são pontos de restauração locais; backups são cópias isoladas em mídia ou storage separado.

No contexto de QEMU+KVM, o backup preserva o estado completo da máquina virtual, incluindo sistema operacional, dados, configurações e histórico de inicialização, permitindo restauração total em caso de perda ou dano do repositório primário.

---

### Por Que Fazer Backup de VMs

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

### Como Fazer Backup: Estratégias

#### 1. Backup a Frio (Máquina Desligada)

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

#### 2. Backup a Quente (Máquina Ligada)

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

#### 3. Backup Incremental

3.1 **Conceito**
   
   a) Primeira execução: cópia completa (full backup)
   
   b) Execuções subsequentes: apenas blocos alterados (delta)
   
   c) Reduz tempo e espaço em storage

3.2 **Implementação**
   
   a) QEMU suporta via `qemu-img convert` com snapshots
   
   b) Ferramentas como `virt-manager` abstraem essa complexidade

---

### Implementação Prática: Backup a Frio com Cópia Direta

#### Passo 1: Desligar a Máquina Virtual

```bash
virsh shutdown win2k25
```

Aguarde a máquina desligar completamente (verificar com `virsh list --all`).

#### Passo 2: Copiar a Imagem QCOW2 para o Destino

```bash
cp ~/libvirt/images/win2k25.qcow2 /media/backup-vm/win2k25.qcow2.backup-$(date +%Y%m%d-%H%M%S)
```

**Explicação:**
- Copia o arquivo de origem para destino com timestamp
- Exemplo de nome gerado: `win2k25.qcow2.backup-20250206-143022`
- Timestamp evita sobrescrita de backups anteriores

#### Passo 3: Verificar Integridade do Backup

```bash
qemu-img info /media/backup-vm/win2k25.qcow2.backup-*
```

Saída esperada mostra tamanho virtual, tamanho real em disco (formato QCOW2) e checksum.

#### Passo 4: Reiniciar a Máquina Virtual

```bash
virsh start win2k25
```

---

### Backup Automatizado com Script e Montagem de Disco

O script abaixo automatiza backup a frio, montando o disco identificado pelo label `backup-vms`, executando a cópia em subpasta dedicada por VM, e desmontando:

```bash
#!/bin/bash

#########################################################
# Script de Backup Automatizado para VMs QEMU+KVM
# Monta disco backup-vms, cria subpasta por VM,
# copia imagem, desmonta
# Uso: ./backup-vm.sh <nome-vm> [destino-label]
#########################################################

set -euo pipefail

# Variáveis
VM_NAME="${1:-win2k25}"
BACKUP_LABEL="${2:-backup-vms}"
SOURCE_DIR="${HOME}/libvirt/images"
MOUNT_POINT="/mnt/backup-temp"
BACKUP_SUBDIR="${MOUNT_POINT}/${VM_NAME}"
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
LOG_FILE="/var/log/backup-vm-${VM_NAME}-${TIMESTAMP}.log"

# Função de logging
log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $*" | tee -a "${LOG_FILE}"
}

# Função de erro
error_exit() {
    log "ERRO: $1"
    cleanup
    exit 1
}

# Função de limpeza (desmonta e remove diretório temporário)
cleanup() {
    log "Iniciando limpeza..."
    if mountpoint -q "${MOUNT_POINT}"; then
        log "Desmontando ${MOUNT_POINT}..."
        sudo umount "${MOUNT_POINT}" || log "AVISO: Falha ao desmontar ${MOUNT_POINT}"
    fi
    if [ -d "${MOUNT_POINT}" ] && [ -z "$(ls -A ${MOUNT_POINT})" ]; then
        rmdir "${MOUNT_POINT}" || log "AVISO: Falha ao remover ${MOUNT_POINT}"
    fi
    log "Limpeza concluída"
}

# Tratamento de sinais para limpeza em caso de interrupção
trap cleanup EXIT INT TERM

# Validações
log "=== Iniciando Backup de VM: ${VM_NAME} ==="

if [ ! -f "${SOURCE_DIR}/${VM_NAME}.qcow2" ]; then
    error_exit "Imagem não encontrada: ${SOURCE_DIR}/${VM_NAME}.qcow2"
fi

# Verificar se VM está ligada e desligar
VM_STATE=$(virsh domstate "${VM_NAME}" 2>/dev/null || echo "undefined")
if [ "${VM_STATE}" = "running" ]; then
    log "VM ${VM_NAME} está em execução. Desligando..."
    virsh shutdown "${VM_NAME}"
    ATTEMPTS=0
    while [ "$(virsh domstate ${VM_NAME} 2>/dev/null)" = "running" ] && [ ${ATTEMPTS} -lt 60 ]; do
        sleep 1
        ATTEMPTS=$((ATTEMPTS + 1))
    done
    if [ $(virsh domstate "${VM_NAME}" 2>/dev/null) = "running" ]; then
        log "AVISO: VM não desligou em tempo. Forçando shutdown..."
        virsh destroy "${VM_NAME}"
    fi
fi

log "VM ${VM_NAME} desligada com sucesso"

# Encontrar disco com label backup-vms
log "Procurando disco com label: ${BACKUP_LABEL}"
DEVICE=$(lsblk -n -o NAME,LABEL | grep "${BACKUP_LABEL}" | awk '{print $1}')

if [ -z "${DEVICE}" ]; then
    error_exit "Disco com label '${BACKUP_LABEL}' não encontrado"
fi

DEVICE="/dev/${DEVICE}"
log "Disco encontrado: ${DEVICE}"

# Criar ponto de montagem se não existir
if [ ! -d "${MOUNT_POINT}" ]; then
    mkdir -p "${MOUNT_POINT}"
    log "Diretório de montagem criado: ${MOUNT_POINT}"
fi

# Montar disco
log "Montando ${DEVICE} em ${MOUNT_POINT}..."
sudo mount "${DEVICE}" "${MOUNT_POINT}" || error_exit "Falha ao montar ${DEVICE}"
log "Disco montado com sucesso"

# Criar subpasta para VM se não existir
if [ ! -d "${BACKUP_SUBDIR}" ]; then
    mkdir -p "${BACKUP_SUBDIR}"
    log "Subpasta criada: ${BACKUP_SUBDIR}"
else
    log "Subpasta já existe: ${BACKUP_SUBDIR}"
fi

# Verificar espaço disponível
AVAILABLE_SPACE=$(df "${MOUNT_POINT}" | tail -1 | awk '{print $4}')
SOURCE_SIZE=$(du -s "${SOURCE_DIR}/${VM_NAME}.qcow2" | awk '{print $1}')

if [ ${SOURCE_SIZE} -gt ${AVAILABLE_SPACE} ]; then
    error_exit "Espaço insuficiente. Necessário: ${SOURCE_SIZE}K, Disponível: ${AVAILABLE_SPACE}K"
fi

log "Espaço verificado: ${AVAILABLE_SPACE}K disponível"

# Executar backup
BACKUP_FILE="${BACKUP_SUBDIR}/${VM_NAME}.qcow2.backup-${TIMESTAMP}"
log "Iniciando cópia: ${SOURCE_DIR}/${VM_NAME}.qcow2 -> ${BACKUP_FILE}"

if cp -v "${SOURCE_DIR}/${VM_NAME}.qcow2" "${BACKUP_FILE}"; then
    log "Cópia concluída com sucesso"
else
    error_exit "Falha na cópia do arquivo"
fi

# Verificar integridade
log "Verificando integridade do backup..."
if qemu-img info "${BACKUP_FILE}" > /dev/null 2>&1; then
    log "Integridade validada com sucesso"
else
    error_exit "Backup corrompido ou inacessível"
fi

# Gerar checksum para verificações futuras
CHECKSUM_FILE="${BACKUP_FILE}.sha256"
sha256sum "${BACKUP_FILE}" > "${CHECKSUM_FILE}"
log "Checksum salvo em: ${CHECKSUM_FILE}"

log "Desmontando disco..."
# Desmontagem será feita pela função cleanup no trap

log "=== Backup concluído com sucesso ==="
log "Arquivo: ${BACKUP_FILE}"
log "Tamanho: $(du -h ${BACKUP_FILE} | cut -f1)"
log "Log completo em: ${LOG_FILE}"

# Reiniciar VM
log "Reiniciando VM ${VM_NAME}..."
virsh start "${VM_NAME}"
log "VM iniciada"
```

---

### Uso do Script

```bash
# Tornar executável
chmod +x backup-vm.sh

# Executar backup (sintaxe: ./backup-vm.sh <nome-vm> [label-disco])
./backup-vm.sh win2k25 backup-vms

# Com parâmetros opcionais
./backup-vm.sh win2k25 backup-vms

# Consultar log
tail -f /var/log/backup-vm-win2k25-*.log
```

---

### Estrutura de Diretórios Resultante

Após executar o script, a organização de backups será:

```
/media/backup-vm/
├── win2k25/
│   ├── win2k25.qcow2.backup-20250206-143022
│   ├── win2k25.qcow2.backup-20250206-143022.sha256
│   ├── win2k25.qcow2.backup-20250207-020000
│   └── win2k25.qcow2.backup-20250207-020000.sha256
├── ubuntuserver/
│   ├── ubuntuserver.qcow2.backup-20250206-143022
│   └── ubuntuserver.qcow2.backup-20250206-143022.sha256
└── debian12/
    ├── debian12.qcow2.backup-20250206-143022
    └── debian12.qcow2.backup-20250206-143022.sha256
```

Cada VM possui sua própria subpasta isolada, facilitando **retenção seletiva**, **políticas de limpeza por VM** e **auditoria granular** de backups corporativos.

---

### Agendamento com Cron

Para backup automático diário às 2h da manhã:

```bash
# Editar crontab
crontab -e

# Adicionar linha:
0 2 * * * /caminho/para/backup-vm.sh win2k25 backup-vms >> /var/log/backup-vm-cron.log 2>&1
```

Para múltiplas VMs em sequência:

```bash
0 2 * * * /caminho/para/backup-vm.sh win2k25 backup-vms >> /var/log/backup-vm-cron.log 2>&1
30 2 * * * /caminho/para/backup-vm.sh ubuntuserver backup-vms >> /var/log/backup-vm-cron.log 2>&1
0 3 * * * /caminho/para/backup-vm.sh debian12 backup-vms >> /var/log/backup-vm-cron.log 2>&1
```

---

### Verificação e Restauração de Backup

#### Listar backups disponíveis

```bash
ls -lh /media/backup-vm/win2k25/
```

Saída esperada:

```
-rw-r--r-- 1 root root 50G Feb  6 14:30 win2k25.qcow2.backup-20250206-143022
-rw-r--r-- 1 root root 128 Feb  6 14:31 win2k25.qcow2.backup-20250206-143022.sha256
-rw-r--r-- 1 root root 50G Feb  7 02:00 win2k25.qcow2.backup-20250207-020000
-rw-r--r-- 1 root root 128 Feb  7 02:01 win2k25.qcow2.backup-20250207-020000.sha256
```

#### Validar checksum antes de restaurar

```bash
sha256sum -c /media/backup-vm/win2k25/win2k25.qcow2.backup-20250206-143022.sha256
```

Saída esperada:

```
win2k25.qcow2.backup-20250206-143022: OK
```

#### Restaurar a partir de backup

```bash
# 1. Parar a VM em produção
virsh destroy win2k25

# 2. Fazer backup da versão atual comprometida (opcional, recomendado)
cp ~/libvirt/images/win2k25.qcow2 ~/libvirt/images/win2k25.qcow2.corrupted-$(date +%Y%m%d)

# 3. Copiar backup para origem
cp /media/backup-vm/win2k25/win2k25.qcow2.backup-20250206-143022 ~/libvirt/images/win2k25.qcow2

# 4. Verificar integridade
qemu-img check ~/libvirt/images/win2k25.qcow2

# 5. Reiniciar VM
virsh start win2k25
```

---

### Manutenção e Limpeza de Backups Antigos

#### Listar backups por antigüidade

```bash
ls -lht /media/backup-vm/win2k25/ | head -10
```

#### Remover backups com mais de 30 dias

```bash
find /media/backup-vm/win2k25/ -name "*.backup-*" -mtime +30 -delete
```

#### Calcular espaço utilizado por VM

```bash
du -sh /media/backup-vm/*/
```

---

### Monitoramento e Alertas

#### Script de verificação de saúde de backups

```bash
#!/bin/bash

# Verificar integridade de todos os backups
for backup in /media/backup-vm/*/*.qcow2.backup-*; do
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



