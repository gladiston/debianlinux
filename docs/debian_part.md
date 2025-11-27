# PARTICIONAMENTO DURANTE A INSTALAÇÃO DO DEBIAN/UBUNTU

## 📋 O que é Particionamento?

**Particionamento** é o processo de dividir seu disco rígido ou SSD em seções lógicas independentes, cada uma com seu próprio sistema de arquivos. Cada partição funciona como um "disco isolado" dentro de seu hardware físico, permitindo organização, otimização e proteção de dados de forma independente.

Durante a instalação do Debian/Ubuntu, você define como o disco será organizado — este é o **ponto mais delicado e crítico** de toda a instalação, pois uma estratégia inadequada pode comprometer performance, segurança e recuperabilidade do sistema.

---

## 🖥️ Esquema de Particionamento Recomendado

A instalação do Debian/Ubuntu não tem grandes mistérios — o ponto mais delicado é mesmo o **particionamento do disco**. Abaixo segue uma sugestão baseada em experiência profissional:

| Sistema | Ponto de Montagem | Rótulo | Tamanho |
|---------|-------------------|--------|---------|
| FAT32 | `/boot/efi` | Nenhum | 1 GB |
| SWAP | Nenhum | Nenhum | Conforme RAM |
| ext4 | `/boot` | #boot | 1 GB |
| ext4 | `/` | #disco1-root | 100 GB |
| ext4 | `/home` | #dados1 | Máximo |

Se irá usar virtualização qemu+kvm e entende dos beneficios do particionamento, recomendo ter um outro disco separado para este proposito, claro que também pode usar uma partição separada, mas ter um ponto de montagem separado como **/var/libvirt** irá lhe garantir maior facilidade de gerenciamento e simplifica as permissões. Neste guia, mais adiante, teremos o procedimento para a virtualização, mas ele irá usar o **/home** porque fica mais fácil para usuários novatos entenderem, mas se for usuário avançado em particionamento e puder, use **/var/libvirt** num ponto ponto de montagem separado.  

---

## 💾 Entendendo a Partição SWAP

### O que é SWAP?

**SWAP** é uma área de armazenamento no disco que funciona como **memória de fuga** ou **memória virtual**. Quando sua memória RAM está completamente preenchida, o sistema operacional move dados menos utilizados da RAM para a partição SWAP no disco, liberando espaço em RAM para novas operações.

**Analogia prática:** Se sua mesa de trabalho (RAM) está cheia de papéis, você move alguns para uma caixa de arquivo (SWAP) embaixo da mesa. Quando precisar deles novamente, você busca de volta na caixa.

### Por que SWAP é Necessário?

Sem SWAP, quando a RAM se esgota completamente, o sistema **trava** e pode apresentar comportamentos impreditíveis — aplicações podem fechar abruptamente ou o sistema congelar. O SWAP oferece um "colchão de segurança", permitindo que o SO continue operacional mesmo em situações de pressão de memória.

### Performance vs Necessidade

> **ALERTA CRÍTICO:** Se sua partição SWAP **está sempre em uso constante**, isto é um sintoma claro de que você precisa **instalar mais memória RAM**. Viver permanentemente com SWAP ativo é desperdício de tempo — disco é **milhares de vezes mais lento** que RAM, comprometendo drasticamente a performance do sistema.

O SWAP deve ser utilizado apenas como **pico ocasional**, não como solução permanente.

### Dimensionamento Recomendado

O tamanho ideal da SWAP depende do seu cenário de uso:

#### Cenário 1: Com Hibernação

Se você deseja usar **hibernação** (suspender o sistema salvando tudo na memória), o SWAP deve ter **no mínimo o tamanho total da RAM**:

- Computador com **16 GB de RAM** e hibernação ativada → SWAP de **16 GB**
- Computador com **32 GB de RAM** e hibernação ativada → SWAP de **32 GB**

Isto ocorre porque o sistema precisa salvar **todo o conteúdo da RAM** na SWAP quando entra em hibernação.

#### Cenário 2: Sem Hibernação (Recomendado para Servidores)

Se você **não utilizará hibernação**, o SWAP pode ser menor:

- **Computador com 8-16 GB de RAM:** SWAP de 4-8 GB
- **Computador com 16-32 GB de RAM:** SWAP de 8-16 GB
- **Computador com 32+ GB de RAM:** SWAP de 4-8 GB (apenas como pico de segurança)

#### Cenário 3: Complementar RAM Insuficiente

Se seu computador possui pouca RAM mas você deseja performance adicional temporária:

- Computador com **8 GB de RAM**, deseja ter **comportamento de 16 GB:** SWAP de **8 GB**
- Computador com **16 GB de RAM**, deseja ter **comportamento de 32 GB:** SWAP de **16 GB**

Neste caso, você terá a RAM física mais o SWAP funcionando em conjunto.

### Conclusão sobre SWAP

- **SWAP é essencial:** Protege contra travamentos quando a RAM se esgota
- **SWAP não é solução:** Se está sempre em uso, você precisa de mais RAM
- **Dimensione corretamente:** Conforme seus hábitos de hibernação e volume de RAM disponível

---

## 🗄️ Sistemas de Arquivos: ext4 vs Btrfs

Se preferir usar o **Btrfs**, o particionamento muda um pouco, nesse caso, `"/"` e `"/home"` ficam na **mesma partição**, que ocupa todo o espaço restante do disco.

O ideal seria ter subvolumes separados para `"/"`, `"/var"` e `"/home"`, mas o instalador padrão do Debian, Ubuntu e da maioria das distros ainda não permitem subvolumes. Dá para fazer manualmente, claro, mas exigiria etapas extras que deixariam este Guia mais complicado — então vamos manter o foco no básico.

O Btrfs é um sistema de arquivos excelente para quem desenvolve software graças ao recurso de **snapshots**, onde é possível recuperar versões anteriores de arquivos apagados ou sobrescritos sem precisar recorrer a backups tradicionais.

Além disso, a **compactação transparente** ajuda a economizar espaço sem perda de desempenho perceptível.

> **ALERTA:** Partições Btrfs não devem ultrapassar 80% de ocupação; acima disso, a performance cai bastante por causa do **Copy-on-Write (CoW)**.

Caso prefira **ext4**, mantenha, se possível, `"/"` e `"/home"` em partições separadas.  

Especialistas em virtualização também recomendam ter `/var/libvirt` em partição ou disco separado; contudo, neste Guia, as VMs ficarão no `$HOME` para simplificar aos iniciantes.

----

[Clique aqui para retornar a página principal](../README.md#particionamento-durante-a-instala%C3%A7%C3%A3o-do-debianubuntu)
