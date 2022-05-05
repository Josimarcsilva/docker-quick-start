# Instalação do Docker no Windows com WSL2

### <a id="docker-engine-docker-nativo-diretamente-instalado-no-wsl2"></a>Docker Engine (Docker Nativo) diretamente instalado no WSL2.

O Docker Engine é o Docker nativo que roda no ambiente Linux e completamente suportado para WSL 2. Sua instalação é idêntica a descrita para as próprias distribuições Linux disponibilizadas no site do [Docker](https://docs.docker.com/engine/install/ubuntu/).

#### Vantagens

* Consome o mínimo de memória necessário para rodar o Docker Daemon (servidor do Docker).
* É mais rápido ainda que com Docker Desktop, porque rodar diretamente dentro da própria instância do WSL2 e não em uma instância separada de Linux.

#### Desvantagens

* Necessário executar o comando ```sudo service docker start``` sempre que o WSL 2 foi reiniciado. Isto não é necessariamente uma desvantagem, mas é bom pontuar, mas isto é um pequeno detalhe e será resolvido futuramente com a inclusão do arquivo /etc/wsl.conf que permitirá incluir comandos para serem executados toda vez que o WSL for reiniciado.
* Se necessitar executar Docker em outra instância do WSL 2, é necessário instalar novamente o Docker nesta instância ou configurar o acesso ao socket do Docker desejado para compartilhar o Docker entre as instâncias.
* Não suporta containers no modo Windows.

## Instalação do WSL 2

### Habilite o WSL no Windows 10

Execute os seguintes comandos no PowerShell em modo administrador:
``` bash
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```
Abra o PowerShell e digite o comando `wsl`, se não funcionar reinicie sua máquina.

### Instale o WSL 2 no Windows 10

Baixe o Kernel do WSL 2 neste link: [http://aka.ms/wsl2kernelmsix64](http://aka.ms/wsl2kernelmsix64) e instale o pacote.

### Atribua a versão default do WSL para a versão 2

A versão 1 do WSL é a padrão no momento, atribua a versão default para a versão 2, assim todas as distribuições Linux instaladas serão já por default da versão 2. Execute o comando com o PowerShell:

``` bash
wsl --set-default-version 2
```

### Escolha sua distribuição Linux no Windows Store

Escolha sua distribuição Linux preferida no aplicativo Windows Store, sugerimos o Ubuntu por ser uma distribuição popular e que já vem com várias ferramentas instaladas por padrão.

Ao iniciar o Linux instalado, você deverá criar um **nome de usuário** que poderá ser o mesmo da sua máquina e uma **senha**, este será o usuário **root da sua instância WSL**.

Parabéns, seu WSL2 já está funcionando:

### (Opcional) Usar Windows Terminal como terminal padrão de desenvolvimento para Windows

Uma deficiência que o Windows sempre teve era prover um terminal adequado para desenvolvimento. Agora temos o **Windows Terminal** construído pela própria Microsoft que permite rodar terminais em abas, alterar cores e temas, configurar atalhos e muito mais.

Instale-o pelo Windows Store.

## O que o WSL 2 pode usar de recursos da sua máquina

Podemos dizer que o WSL 2 tem acesso quase que total ao recursos de sua máquina. Ele tem acesso por padrão:

* A todo disco rígido.
* A usar completamente os recursos de processamento.
* A usar 80% da memória RAM disponível.
* A usar 25% da memória disponível para SWAP.

Isto pode não ser interessante, uma vez que o WSL 2 pode usar praticamente todos os recursos de sua máquina, mas podemos configurar limites.

Crie um arquivo chamado `.wslconfig` na raiz da sua pasta de usuário `(C:\Users\<seu_usuario>)` e defina estas configurações:

```txt
[wsl2]
memory=8GB
processors=4
swap=2GB
```

Estes são limites de exemplo e as configurações mais básicas a serem utilizadas, configure-os às suas disponibilidades.
Para mais detalhes veja esta documentação da Microsoft: [https://docs.microsoft.com/pt-br/windows/wsl/wsl-config#wsl-2-settings](https://docs.microsoft.com/pt-br/windows/wsl/wsl-config#wsl-2-settings).

Para aplicar estas configurações é necessário reiniciar as distribuições Linux, então sugerimos executar no PowerShell o comando: `wsl --shutdown` (Este comando vai desligar todas as instâncias WSL 2 ativas e basta abrir o terminal novamente para usa-la já com as novas configurações).

## <a id="instalar-o-docker-com-docker-engine-docker-nativo"></a>1 - Instalar o Docker com Docker Engine (Docker Nativo)

A instalação do Docker no WSL 2 é idêntica a instalação do Docker em sua própria distribuição Linux, portanto se você tem o Ubuntu é igual ao Ubuntu, se é Fedora é igual ao Fedora. A documentação de instalação do Docker no Linux por distribuição está [aqui](https://docs.docker.com/engine/install/), mas vamos ver como instalar no Ubuntu.


> **Quem está migrando de Docker Desktop para Docker Engine, temos duas opções**
> 1. Desinstalar o Docker Desktop.
> 2. Desativar o Docker Desktop Service nos serviços do Windows. Esta opção permite que você utilize o Docker Desktop, se necessário, para a maioria dos usuários a desinstalação do Docker Desktop é a mais recomendada.
>Se você escolheu a 2º opção, precisará excluir o arquivo ~/.docker/config.json e realizar a autenticação com Docker novamente através do comando "docker login"

> **Se necessitar integrar o Docker com outras IDEs que não sejam o VSCode**
>
> O VSCode já se integra com o Docker no WSL desta forma através da extensão Remote WSL ou Remote Container.
> 
> É necessário habilitar a conexão ao servidor do Docker via TCP. Vamos aos passos:
> 1. Crie o arquivo /etc/docker/daemon.json: `sudo echo '{"hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]}' > /etc/docker/daemon.json`
> 2. Reinicie o Docker: `sudo service docker restart`
> 
> Após este procedimento, vá na sua IDE e para conectar ao Docker escolha a opção TCP Socket e coloque a URL `http://IP-DO-WSL:2375`. Seu IP do WSL pode ser encontrado com o comando `cat /etc/resolv.conf`.
> 
> Se caso não funcionar, reinicie o WSL com o comando `wsl --shutdown` e inicie o serviço do Docker novamente.


Instale os pré-requisitos:

```
sudo apt update && sudo apt upgrade
sudo apt remove docker docker-engine docker.io containerd runc
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release

```

Adicione o repositório do Docker na lista de sources do Ubuntu:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Instale o Docker Engine

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

```

Dê permissão para rodar o Docker com seu usuário corrente:

```
sudo usermod -aG docker $USER
```
```
sudo chmod 666 /var/run/docker.sock
```
Instale o Docker Compose:

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

Inicie o serviço do Docker:

```
sudo service docker start
```

Este comando acima terá que ser executado toda vez que Linux for reiniciado. Se caso o serviço do Docker não estiver executando, mostrará esta mensagem de erro:

```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

## Dicas e truques básicos com WSL 2

* A performance do WSL 2 está em se executar tudo dentro do Linux, por isso evite executar seus projetos com ou sem Docker do caminho `/mnt/c`, pois você perderá performance.
* Para abrir o terminal do WSL basta digitar o nome da distribuição no menu Iniciar ou executar `C:\Windows\System32\wsl.exe`.
* O sistema de arquivos do Windows 10 é acessível em `/mnt`.
* É possível acessar o sistema de arquivos do Linux pela rede do Windows, digite `\\wsl$` no Windows Explorer.
* É possível acessar uma pasta no Windows Explorer digitando o comando ```explorer.exe .```.
* É possível abrir uma pasta ou arquivo com o Visual Studio Code digitando o comando ```code . ou code meu_arquivo.ext```.
* Incrivelmente é possível acessar executáveis do Windows no terminal do Linux executando-os com .exe no final (não significa que funcionarão corretamente).
* É possível executar algumas aplicações gráficas do Linux com WSL 2. Leia este tutorial: [https://medium.com/@dianaarnos/aplica%C3%A7%C3%B5es-gr%C3%A1ficas-no-wsl2-e0a481e9768c](https://medium.com/@dianaarnos/aplica%C3%A7%C3%B5es-gr%C3%A1ficas-no-wsl2-e0a481e9768c).
* Execute o comando ```wsl -l -v``` com o PowerShell para ver as versões de Linux instaladas e seu status atual(parado ou rodando).
* Execute o comando ```wsl --shutdown``` com o PowerShell para desligar todas as distribuições Linux que estão rodando no momento (ao executar o comando, as distribuições do Docker também serão desligadas e o Docker Desktop mostrará uma notificação ao lado do relógio perguntando se você quer iniciar as distribuições dele novamente, se você não aceitar terá que iniciar o Docker novamente com o ícone perto do relógio do Windows).
* Execute com o PowerShell o comando ```wsl --t <distribution name>``` para desligar somente uma distribuição Linux específica.
* Se verificar que o WSL 2 está consumindo muitos recursos da máquina, execute os seguintes comandos dentro do terminal WSL 2 para liberar memória RAM:
```bash
echo 1 | sudo tee /proc/sys/vm/drop_caches
```
* Acrescente `export DOCKER_BUILDKIT=1` no final do arquivo .profile do seu usuário do Linux para ganhar mais performance ao realizar builds com Docker. Execute o comando `source ~/.profile` para carregar esta variável de ambiente no ambiente do seu WSL 2.

## Comandos úteis

Inicia o serviço Docker

```
sudo service docker start
```


Lista todos os container ativos

```
docker ps
```

Lista todos os containers (ativos ou parados)

```
docker ps -a
```

Remove todos os containers parados 

```
docker container prune
```

Cria uma container nomeado sem travar o terminal

```
docker run --name my-redis -d redis
```

## Créditos

[https://github.com/codeedu/wsl2-docker-quickstart](https://github.com/codeedu/wsl2-docker-quickstart)


