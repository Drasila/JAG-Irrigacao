# JAG-Irrigacao
Sistema eficiente de irrigação doméstica baseado em Python e feito para rodar no Raspberry Pi

Este programa é composto de um script escrito 100% em python e um arquivo de configuração, ele foi feito rodar em um raspberry pi e ser executado uma vez por dia por meio da função cron incluída no kernel do sistema operacional linux. Ao ser executado, o programa obtém dados climáticos da instituição Open Weather por meio de uma API, baseando-se nos dados obtidos, mais especificamente, no índice pluviométrico das últimas 24 horas, o programa decide se deve ou não ativar uma porta GPIO de 3.3V no raspberry pi que por padrão é dormente e não emite nenhuma voltagem. A ativação da porta 3.3V energiza um solenoide, que faz com que dois caminhos elétricos se conectem manualmente, esta peça é conhecida como relé, ao se unificarem, estes caminhos elétricos permitem que uma corrente de 12V alcance um segundo solenoide, este que faz parte de uma válvula que controle o fluxo da água. Ao ser ativada, a válvula solenoide permite o fluxo de água de um compartimento de água que coleta água da chuva até uma encanação que irá irrigar as plantas.

Uma chave de API do Open Weather é necessária para que o programa funcione, é possível conseguir uma com um limite de chamadas mensais por meio do seguinte link:
https://openweathermap.org/api
Mais especificamente, a API usada é a de weather history, descrita no link abaixo:
https://openweathermap.org/api/one-call-api#history
Além dos módulos que vem instalados por padrão com o python, o programa faz utilização dos módulos requests e configparser, que precisam ser instalados a parte. Recomendamos que o raspberry pi esteja rodando raspbian 32bits com uma interface gráfica, o sistema operacional próprio do raspberry pi. Os comandos abaixo deixarão o raspberry pi pronto para rodar JAG-Irrigação. os mesmos comandos supostamente funcionariam com ubuntu e debian, porém foram testados apenas para raspbian.

sudo apt-get update && sudo apt-get upgrade
sudo apt-get install python-pip
pip install requests
pip install configparser

O primeiro comando atualiza o banco de dados do gerenciador de pacotes apt e faz o downloads de todos os pacotes com versões mais recentes logo em seguida. O segundo comando instala pip, que é um gerenciador de pacote específico de python que auxilia na instalação de módulos python. O terceiro e o quarto comando instalam respectivamente, os módulos resquests e configparser.

Copie a pasta jag-irrigaçao para o seu diretório home, abra o arquivo config dentro de jag-irrigacao e configure-o de acordo com sua necessidade. Antes de configurar o arquivo crontab, é uma boa ideia testar se o programa está funcionando, abra uma janela terminal de dentro da pasta jag-irrigacao e digite o seguinte comando:

python executar_irrigacao test

Se você ver uma mensagem como a mensagem abaixo, as configurações estão corretas e o programa está pronto para rodar:

API está funcionando, pluviosidade das últimas 24 horas=0.000000

Caso você receba uma mensasgem diferente, verifique sua conexão com a internet, a chave da API e todos os outros parâmetros inseridos no arquivo config.

Para rodar o programa e pular o critério de ter chovido ou não nas últimas 24 horas, rode o seguinte comando:

sudo python executar-irrigacao.py force

Por padrão, o programa registra suas atividades em um arquivo de texto localizado em /var/log/jag-irrigacao/irrigacao.log, este caminho pode ser modificado no arquivo config. É importante notar que diferentes raspberry pis possuem diferentes numerações para cada uma de suas portas GPIO, este programa foi desenvolvido usando o raspberry pi 4B, é importante ajustar o número da porta GPIO no arquivo config de acordo com a versão do seu raspberry pi.

Se por qualquer motivo você precisar resetar o estado inicial das portas GPIO, ou seja, fazer quem que elas parem de emitir qualquer sinal elétrico, basta rodar o seguinte comando:

sudo python executar-irrigacao.py reset

Instalando o crontab

Agora que o programa já foi testado e está funcionando, o último passo é configurar o arquivo crontab que executará "executar-irrigação.py" de acordo com a rotina configurada. Um crontab (abreviado cron) é um simples arquivo de texto que é usado pelo Linux para rodar tarefas em intervalos específicados. Neste exemplo configuraremos o aquivo crontab para executar o programa "executar-irrigacao.py" às 06:00 e 18:00 todos os dias.
Abra o arquivo run.crontab com um editor de texto, se estiver usando um terminal, eu recomendo nano, portanto digite:

nano run.crontab

O formato básico do crontab é da seguinte maneira (note que as linhas que começam com um "#" são apenas comentários):

# minuto   hora    dia do mes     mes    dia da semana      Comando
# (0-59)  (0-23)    (1-31)       (1-12)     (0-6)
    *       *         *            *          *             xyz comando

Decida o intervalo de tempo em que você quer que os comandos sejam executados e substitua os asteriscos com os devidos minutos, horas, dias do mês, etc. Para executar nosso programa às 06:00 e 18:00 todos os dias, o arquivo run.crontab ficará da seguinte maneira:

# minuto   hora    dia do mes     mes    dia da semana      Comando
# (0-59)  (0-23)    (1-31)       (1-12)     (0-6)
    0      6,18       *            *          *        /usr/bin/python2.7 /home/pi/jag-irrigacao/executar-irrigacao.py
@reboot                                                /usr/bin/python2.7 /home/pi/jag-irrigacao/executar-irrigacao.py reset > /var/log/jag-irrigacao/cron_reboot_erros.log 2>&1

O "6,18" indica o cron para executar o comando às 0600 e 1800 horário local todos os dias. Perceba que nós incluimos o caminho completo para o executável do python e também para o nosso programa executar-irrigacao.py, caso você tenha colocado sua pasta jag-irrigação em um caminho diferente, não esqueça de mudar no cron. É importante não usar caminhos relativos na configuração do cron pois ele roda em um ambiente mínimo e é bem provável que não reconhecerá os caminhos relativos, sempre use caminhos absolutos.
Note que o segundo comando é executado toda vez que o sistema operacional é reiniciado, este comando seta as portas GPIO para 0V, garantindo que a voltagem nunca seja positiva no startup e que a válvula não abra acidentalmente, ele também gera logs. Há uma rotina de reinicialização no final do nosso programa para garantir que o raspberry pi reinicie após cada irrigação, isso previne erros ao longo prazo.

O exemplo a seguir irá executar o programa ao meio dia de toda segunda-feira, quarta-feira e sexta-feira.

# minuto   hora    dia do mes     mes    dia da semana      Comando
# (0-59)  (0-23)    (1-31)       (1-12)     (0-6)
    0      12         *            *       1,3,5        /usr/bin/python2.7 /home/pi/jag-irrigacao/executar-irrigacao.py
@reboot                                                /usr/bin/python2.7 /home/pi/jag-irrigacao/executar-irrigacao.py reset > /var/log/jag-irrigacao/cron_reboot_erros.log 2>&1

O seguinte website pode te auxiliar a monter a sua rotina do crontab (note que existem outros além deste):

https://crontab-generator.org/

Após ter seu arquivo run.crontab configurado e pronto, acesse o terminal e digite o seguinte comando:

sudo crontab run.crontab

O crontab reiniciará junto com seu raspberry pi, para verificar se ele está rodando no background, use o seguinte comando:

sudo crontab -l

Se você ver a rotina que você definiu no arquivo run.crontab listada, tudo está em dia, caso contrário, instale o crontab novamente, caso deseje remover o crontab atual, basta digitar:

sudo crontab -r

Isso é tudo, divirta-se!
