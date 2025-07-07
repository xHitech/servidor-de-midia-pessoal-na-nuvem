# servidor-de-midia-pessoal-na-nuvem
Documentação do projeto de criação de um servidor de mídia pessoal 100% na nuvem, utilizando VPS, Rclone e Komga, com automação via Systemd.

1. O Problema
A motivação inicial para este projeto foi resolver uma série de problemas práticos. Eu precisava de acesso constante (24/7) à minha biblioteca de mídia de 2TB no Google Drive, sem depender de um PC gamer para isso. Manter o computador principal ligado o tempo todo gerava ruído indesejado, tinha um alto consumo de energia e, o mais crítico, deixava meu hardware caro vulnerável à instável rede elétrica da minha cidade.

2. A Arquitetura da Solução

A solução escolhida foi uma arquitetura 100% na nuvem, composta por:
Infraestrutura: Um VPS (Servidor Virtual Privado) rodando Ubuntu Server.
Armazenamento: Rclone, configurado para "montar" o Google Drive como um sistema de arquivos local no servidor, servindo como uma ponte de acesso aos dados sem ocupar espaço em disco local.

Aplicações:

Komga: Um servidor de mídia open-source especializado para quadrinhos e mangás.
Plex: Uma das plataformas de servidor de mídia mais populares do mundo, utilizada aqui com foco no streaming de uma biblioteca de áudio em formato FLAC.
Automação: Systemd, o gerenciador de sistemas do Linux, para garantir que todos os serviços (Rclone, Komga, Plex) iniciem automaticamente com o sistema e se recuperem de falhas.

3. Tecnologias Utilizadas

Cloud: VPS (IaaS)
OS: Ubuntu Server 20.04 LTS
Core: Rclone, FUSE3
API: Google Drive API, OAuth 2.0
Serviços: Komga (Java), Plex Media Server
Automação: Systemd
Firewall: UFW

4. Guia de Configuração e Desafios
Esta seção detalha o passo a passo da jornada, incluindo os problemas encontrados e como foram resolvidos, demonstrando um ciclo completo de troubleshooting.

4.1. Configuração do Rclone e da API do Google
A configuração inicial do Rclone exigiu a criação de credenciais OAuth 2.0 no Google Cloud Platform. O primeiro desafio foi um erro de invalid_client retornado pelo Google, que foi solucionado ao garantir a integridade do Client ID/Secret e realizar a autorização de forma manual, um passo necessário para ambientes de servidor sem interface gráfica.

4.2. O Desafio do FUSE (Filesystem in Userspace)
Mesmo com o Rclone configurado, a montagem do drive falhava com o erro fusermount3 not found. A solução envolveu instalar o pacote fuse3, editar o arquivo /etc/fuse.conf para permitir a opção user_allow_other e adicionar o usuário ao grupo fuse (sudo usermod -aG fuse [usuário]).

4.3. Otimização da Montagem e Resolução de Cache
A varredura inicial das bibliotecas era extremamente lenta. Para resolver isso, a montagem do Rclone foi otimizada com flags de performance, como --dir-cache-time, --vfs-cache-mode full e --buffer-size. Posteriormente, um problema de cache que impedia a atualização de metadados foi resolvido habilitando o "controle remoto" do rclone com a flag --rc e utilizando o comando rclone rc vfs/refresh para forçar a atualização dos diretórios sob demanda.

4.4. Instalação e Automação dos Servidores de Mídia
Para garantir a robustez do sistema, foram criados serviços no systemd para o rclone, Komga e Plex. A dependência entre os serviços foi garantida com a diretiva After=rclone-mount.service, assegurando que o drive sempre estaria disponível antes das aplicações iniciarem. A instalação do Plex exigiu um passo adicional: a criação de um túnel SSH (ssh -L 8888:localhost:32400) para realizar a configuração inicial de forma segura em um servidor remoto.

5. Arquivos de Configuração
Exemplos dos arquivos de serviço do Systemd criados para este projeto.


plexmediaserver.service (gerado automaticamente pela instalação do Plex)

6. Resultado Final!
[Screenshot 2025-07-07 185531](https://github.com/user-attachments/assets/40bbf9e1-d90a-44ba-bf19-6e7ab0eea142)

O resultado é um servidor de mídia estável, silencioso, de baixo consumo (fora de casa), acessível de qualquer lugar e totalmente autônomo, com os serviços iniciando automaticamente na ordem correta e se recuperando de falhas.
