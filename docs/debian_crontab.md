# CRONTAB - AGENDADOR DE TAREFAS
O **crontab** é o agendador de tarefas do Linux. Ele permite que comandos ou scripts sejam executados automaticamente em horários ou intervalos definidos, sem intervenção do usuário.  
É extremamente útil para tarefas recorrentes, como backups, limpeza de temporários, sincronização de dados e atualizações.

O daemon do cron (**crond**) roda em segundo plano e verifica, minuto a minuto, se há tarefas programadas.  
Cada usuário tem seu **próprio crontab**; além disso, existe um **crontab do sistema** em `/etc/crontab` (e arquivos em `/etc/cron.d/`).

Para editar **o crontab do seu usuário** (formato de 5 campos), use:
```bash
crontab -e
```

> **Atenção (formato):**  
> - **Crontab de usuário** (`crontab -e` e também `sudo crontab -e`, que edita o crontab **do usuário root**) usa **5 campos** de tempo e **não** tem coluna “nome-usuario”.  
> - **Crontab do sistema** (`/etc/crontab` e arquivos em `/etc/cron.d/`) usa **6 campos**, incluindo a coluna **nome-usuario**.

Se quiser editar **o crontab do sistema**, edite o arquivo diretamente:
```bash
sudo editor /etc/crontab
```

Mesmo que não use o crontab agora, recomendo colar um **cabeçalho de referência**.

**Modelo para crontab de usuário (5 campos):**
```crontab
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""

# .---------------- minuto (0 - 59)
# |  .------------- hora (0 - 23)
# |  |  .---------- dia do mês (1 - 31)
# |  |  |  .------- mês (1 - 12) OU jan,fev,mar,abr ...
# |  |  |  |  .---- dia da semana (0 - 6) (domingo=0 ou 7) OU dom,seg,ter,qua,qui,sex,sab
# |  |  |  |  |
# *  *  *  *  *  comando a ser executado
#
# Ex.: executar backup todos os dias às 02:30 e registrar log
# 30 2 * * * /usr/local/bin/backup-diario.sh >> /var/log/backup.log 2>&1
```

**Modelo para `/etc/crontab` ou `/etc/cron.d/*` (6 campos, com usuário):**
```crontab
# /etc/crontab: system-wide crontab
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""

# .---------------- minuto (0 - 59)
# |  .------------- hora (0 - 23)
# |  |  .---------- dia do mês (1 - 31)
# |  |  |  .------- mês (1 - 12)
# |  |  |  |  .---- dia da semana (0 - 6)
# |  |  |  |  |
# *  *  *  *  *  nome-usuario  comando a ser executado
#
# Ex.: executar backup todos os dias às 02:30 como root
# 30 2 * * * root /usr/local/bin/backup-diario.sh >> /var/log/backup.log 2>&1
```

Por que deixar as linhas acima? Para lembrar rapidamente o **formato** correto (com ou sem coluna de usuário) conforme o tipo de crontab que você estiver editando.

- **SHELL**: usar `/bin/sh` evita carregar perfis com variáveis/aliases que possam interferir.  
- **PATH**: necessário pois o `sh` costuma ter PATH mínimo.  
- **MAILTO**: em servidores com MTA/SMTP, você pode apontar um e-mail para receber a saída/erros dos jobs.

Para **listar** agendamentos:
```bash
crontab -l           # lista do seu usuário
sudo crontab -l      # lista do usuário root
```

**Exemplo (crontab do sistema)**: desligar o computador às 02:00 (independente de login):
```crontab
# Desligar o computador as 02h00 da madrugada
0 2 * * * root /usr/sbin/shutdown -h now
```

**Exemplo (crontab de usuário)**: lembrete para se levantar e beber água **das 10h às 18h, a cada 2 horas**:
```crontab
# Enviar lembrete gráfico (KDE, GNOME, etc.)
0 10-18/2 * * * /usr/bin/notify-send "Lembrete: Levante-se um pouco e beba água!"
# (opcional) em alguns ambientes pode ser necessário também:
# 0 10-18/2 * * * DISPLAY=:0 DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus /usr/bin/notify-send "Lembrete..."
```

O uso do **crontab** aqui vale para todas as distribuições Linux, inclusive em desktops.  
GNOME, KDE e outras DEs possuem utilitários visuais para agendar tarefas, mas, neste HowTo, priorizamos o terminal por ser a forma mais consistente e portátil de configurar.
