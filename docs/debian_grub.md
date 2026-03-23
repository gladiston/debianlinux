
# **Mais segurança com o GRUB**

Este guia apresenta práticas simples para aumentar a segurança do boot do seu sistema Linux, especialmente em cenários onde atualizações de kernel podem causar instabilidade.

O **Debian** é conhecido por ser extremamente conservador e cuidadoso com atualizações, reduzindo bastante o risco de problemas após upgrades.
Já o **Ubuntu** e suas distribuições derivadas (Kubuntu, Xubuntu, Pop!_OS, etc.) tendem a incluir versões mais recentes de kernel com maior frequência — o que é ótimo para compatibilidade de hardware, mas pode ocasionalmente causar falhas no boot.

Para evitar dores de cabeça, recomendamos duas medidas fundamentais:

1. Ativar um tempo de espera no GRUB (padrão no Debian, mas não no Ubuntu)
2. Saber como remover kernels problemáticos e evitar reinstalação automática

---

## 1. Habilitar temporizador no GRUB (5 segundos)

### Por que isso é importante?

O menu do GRUB permite escolher versões anteriores do kernel caso algo dê errado.
No entanto, no Ubuntu esse menu geralmente fica oculto e inicia automaticamente.

Sem tempo de espera, você pode ficar preso em um sistema que não inicia corretamente.

---

### Como configurar

Edite o arquivo de configuração:

```bash
sudo editor /etc/default/grub
```

Ajuste ou adicione as linhas:

```
GRUB_TIMEOUT=5
GRUB_RECORDFAIL_TIMEOUT=5
```

---

### Aplicar alterações

```bash
sudo update-grub
```

---

### Resultado esperado

Agora, ao iniciar o sistema:

* O menu do GRUB será exibido por 5 segundos
* Será possível escolher kernels antigos facilmente
* Em caso de falha, o GRUB continuará exibindo o menu

---

## 2. Remover kernel problemático (Ubuntu e derivados)

### Cenário

Após uma atualização, o sistema para de iniciar corretamente.

Solução: iniciar com um kernel antigo e remover o kernel defeituoso.

---

### 2.1 Identificar kernels instalados

```bash
dpkg -l | grep '^ii' | grep linux-image
```
Uma saida como esta será exibida:
```text
ii  linux-image-6.17.0-14-generic                            6.17.0-14.14~24.04.1                        amd64        Signed kernel image generic
ii  linux-image-6.17.0-19-generic                            6.17.0-19.19~24.04.2                        amd64        Signed kernel image generic
ii  linux-image-generic-hwe-24.04                            6.17.0-19.19~24.04.2                        amd64        Generic Linux kernel image
```


---

### 2.2 Verificar kernel em uso

```bash
uname -r
```

Atenção: nunca remova o kernel atualmente em uso.

---

### 2.3 Remover kernel problemático
Em nosso exemplo, demos boot com o kernel mais antigo, isto é, **linux-image-6.17.0-14-generic** para que seja possível remover o mais recente **linux-image-6.17.0-19-generic** assim:

```bash
sudo apt purge linux-image-6.x.x-xx-generic linux-headers-6.x.x-xx-generic
```

---

### 2.4 Atualizar GRUB

```bash
sudo update-grub
```

---

## 3. Evitar reinstalação automática do kernel (HOLD)

### Por que isso é necessário?

No Ubuntu, kernels são gerenciados por meta-pacotes como:

```
linux-image-generic
linux-image-generic-hwe-24.04
```

Esses pacotes sempre puxam versões mais novas automaticamente.

Se você não fizer nada, o kernel problemático pode voltar na próxima atualização.

---

### 3.1 Identificar meta pacote

```bash
dpkg -l | grep linux-image-generic
```

---

### 3.2 Congelar atualização (recomendado)

```bash
sudo apt-mark hold linux-image-generic linux-image-generic-hwe-24.04
```

---

### 3.3 Conferir status

```bash
apt-mark showhold
```

---

### Resultado esperado

* O sistema não instalará novos kernels automaticamente
* Você terá controle manual sobre atualizações
* Reduz drasticamente o risco de quebrar o sistema

---

## 4. Considerações importantes

Debian:

* Já possui comportamento mais seguro por padrão
* Atualizações mais estáveis

Ubuntu:

* Mais atualizado, porém com maior risco
* Exige controle manual em ambientes críticos

Boas práticas:

* Sempre manter pelo menos 1 kernel antigo funcional
* Nunca remover o kernel em uso
* Testar novos kernels antes de confiar totalmente

---

## Conclusão

Com apenas duas configurações simples:

* Tempo de espera no GRUB
* Controle de atualização de kernel

Você transforma seu sistema em um ambiente muito mais seguro e previsível.

Essas práticas são especialmente recomendadas para:

* servidores
* máquinas de produção
* ambientes de desenvolvimento críticos

----

[Clique aqui para retornar a página principal](../README.md#MAIS SEGURANÇA COM O GRUB)
