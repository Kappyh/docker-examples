Estudos sobre Docker

-> Usa uma comando a menos ao montar o container, em relação a VM, usando
os recursos do sistema operacional host alocado separadamente, ao invés de instalar um sistema operacional no container guest.

-> o LXC é a tecnologia base do docker, porém o docker gerencia de forma mais fácil os containers e ainda disponibiliza uma API REST para gerenciar os containers e um CLI.

---------
-> Container
- Processo segregado usando kernel da máquina host
- Sistema de arquivos criados a partir de uma imagem
- Imagem leve e portátil no qual aplicações são executadas
- Encapsula todos os binários e bibliotecas necessários de um App
- Algo entre um **chroot** e uma **VM**
- Tentar ser o mais minimalista possível na função do container: Um container para aplicação, um container para banco de dados

--------
-> O que são as imagens docker
- Modelo de sistema de arquivos para criar um container
- Imagens são criadas através de um processo chamado build(o commit não é uma boa prática, pois é difícil rastrear o que foi alterado)
- São armazenadas em repositórios no Registry
- Possui uma ou mais camadas (layers)
- Cada camada representa uma mudança no sistema de arquivos (imagem intermediaria)
- Dividir em camadas facilita o reuso
- A junção de camadas(layers) forma a imagem
- É possível usar layers de outras imagens em uma imagem
- A imagem é um sistema de arquivos somente-leitura para criar containers

--------
Imagem X Containers

Comparando como orientação a objeto

Imagem === classe
Container === objeto

--------
Instalação Linux

https://www.vivaolinux.com.br/dica/Docker-varrundockersock-connect-permission-denied-Resolvido

--------
Testando a instalação

```
docker container run hello-world
```

Mostrar containers parados

```
docker container ps -a
docker container ls -a
```

O método run do docker **sempre** cria novos containers

```
docker container run -it debian bash
```

a flag -it entra no container para interagir no terminal dele após cria-lo.

---------------
Criando container com nome

A flag --name cria um container com nome, não é possível criar 2 com o mesmo nome
```
docker container run --name mydeb -it debian bash
```

Executando um container com nome + modo interativo

```
docker container start -ai mydeb
```

Mapeamento de portas

Flag -p portaHost(local):portaGuest(container) . Exemplo

```
docker container run -p 8080:80 nginx
```

No exemplo acima ao acessar o endereço localhost:8080 da maquina local irá apontar para o container na porta 80, onde há uma instalação do servidor nginx


Mapeamento de volumes(diretorios)

A flag -v indica que será mapeado uma pasta da máquina host: pasta guest(docker)

```
docker container run -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx
```

No exemplo acima o docker cria uma refêrencia do $(pwd) -> pasta onde está executando o comando
e aponta para uma referencia dentro do container, ou seja, ao invés de rodar o que está na pasta /usr/share/nginx/html do container, vai refletir o que está na pasta da máquina host.

Rodando container em background( Modo Daemon )

Flag -d roda o container em background, ou seja, não fica ocupando uma sessão no terminal que pode ser parada com ctrl+c
```
docker container run -d --name ex-daemon-basic -p 8080:80 -v $(pwd)/html:/usr/share/nginx/html nginx

```

Start, Stop e Restart Container

Você pode executar estes comandos passando o nome do container ou o container ID:

```
docker container start [nome-do-container ou container-id]
docker container stop [nome-do-container ou container-id]
docker container restart [nome-do-container ou container-id]
```

O container ID é exibido no comando docker ps ou quando o container é criado.
Exemplo de start
```
docker container start server-meu-site
docker container start 04f664525d82
```

Comandos (resumo até agora)

Listagem de containers

```
// sintaxe nova - listagem
docker container ls
docker container list
docker container ps

// sintaxe antiga - listagem
docker ps

// Listagem com todos os containers, inclusive parados

docker container ls -a
docker container list -a
docker container ps -a

// Listagem com todos os container. inclusive parados(forma antiga)
docker ps -a

// Exibindo logs (container deve estar sendo executado)
docker container logs [nome-do-container ou Container ID]

// Configurações em formato JSON
docker container inspect [nome-do-container ou Container ID]

// Executando um comando dentro do container (exec)
docker container exec [nome-do-container ou Container ID] [comando-para-executar]

// Exemplo do comando acima
docker container exec ex-daemon-basic uname -or
(mostra a versão do kernel do Linux)


// Removendo imagens, volumes e containers

docker container rm [nome-do-container ou Container ID]
docker volume rm [nome-do-container ou Container ID]
docker image rm [nome-do-container ou Container ID]


// Observações sobre os comandos de remoção:

// A sintaxe antiga do comando era docker rmi para remoção de imagens
// Não se pode remover imagens ou volumes sendo usados por um container,
// é necessário remover o container primeiro


```

### Construção de imagens do docker

Comandos úteis (documentação do CLI).

Mostra a documentação dos respectivos comandos

```
docker container --help
docker image --help
docker volume --help
```

Toda imagem tem hash char256, porém é possível nomear as imagens

Listando imagens
```
docker image ls
```

Criando uma tag a partir de uma imagem já listada

```
docker image tag [nome-imagem-original] [nome-da-tag]
```

Fazendo dessa forma a nova tag é listada junto com a imagem original.


#### Gerenciando imagens(comandos básicos)

```
// baixa uma imagem do hub
docker image pull [nome-da-imagem]

// documento da imagem
docker image inspect [nome-da-imagem]

// gera uma imagem
docker image build

// publicação da imagem do HUB ou registry local
docker image push
```

Diferença entre docker registry e docker hub

docker registry pode ser inserido no ambiente da empresa pra registro e resgate de  imagens. Disponibiliza uma API, pode ser publico ou privado.

docker hub, contém o registry e o uma interface gráfica, é uma plataforma SaaS, além do que é mantido oficialmente pelo docker e diversas empresas e imagens oficiais. (atualizações frequentes)

Eles não são a mesma coisa. Registry é uma API, o HUB é um SaaS (publico)

#### Criando a primeira imagem

Dentro de um diretório crie um arquivo com nome `Dockerfile`
Descreva a imagem que o arquivo vai usar como no exemplo
```
FROM nginx:latest

RUN echo '<h1>Hello world! </h1>' > /usr/share/nginx/html/index.html
```
Explicando o código acima, primeira layer da imagem possui a imagem do nginx
na última versão(não recomendado para produção).
Depois adicionou uma mensagem de Hello World no arquivo index da pasta raíz que é criada pela layer do nginx

-> Build da imagem
Para buildar a imagem é necessário rodar o comando, e é necessário estar dentro do próprio diretório, a flag -t serve para nomear a imagem com uma tag
```
docker image build -t ex-simple-build .
```

Caso não queira rodar a partir do diretório onde está o arquivo precisa remover o `.` do final do comando e apontar a pasta onde se encontra o Dockerfile que será buildado

Com isso a imagem está criada, rode o `docker image ls` para confirar se foi criado e rode o comando para criar um container a partir dessa imagem:
```
docker container run -p 8080:80 ex-simple-build
```

Acesse agora o endereço `http://localhost:8080` e veja a mensagem do arquivo index.html do container.


#### Dockerfile com argumentos

Cada comando gera uma nova layer

```
FROM debian

LABEL maintainer 'Gabriela Mendes'

  

ARG S3_BUCKET=files

ENV S3_BUCKET=${S3_BUCKET}

```

O comando para build permanece o mesmo, porém você pode passar modificar
o argumento  S3_BUCKET e sobrescrever o valor padrão files

```
docker image build --build-arg S3_BUCKET=myapp -t ex-build-arg
```

Na hora de gerar o container você pode mostrar o valor do argumento pelo bash, para confirmar que o valor na imagem foi alterado
```
docker container run ex-build-arg bash -c 'echo $S3_BUCKET'
```

É possível também imprimir o valor do label definido no Dockerfile com o comando
```
docker image inspect --format="{{index .Config.Labels \"maintainer\"}}"
```

#### Dockerfile com copy

Arquivo Dockerfile
```
FROM nginx:latest

LABEL maintainer 'Gabriela Mendes \- ViralTech'

  

RUN echo '<h1>Sem conteudo<h1>' \> /usr/share/nginx/html/conteudo.html

COPY \*.html /usr/share/nginx/html/
```

Na linha do RUN echo está sendo criado um arquivo conteudo.html no diretorio de pastas do nginx 
Na linha COPY o conteúdo do diretorio atual onde se encontra o Dockerfile com extensão .html está sendo copiado para a pasta dos arquivos do nginx dentro do container Docker.

```
// Para gerar a imagem (dentro do diretorio)
docker image build -t ex-build-copy .

// Para gerar container a partir da imagem criada
docker container run -p 8080:80 ex-build-copy

```


#### Acessando volumes a partir de um container 

Exemplo do `Dockerfile` no endereço do Github

Crie a imagem a partir do Dockerfile que se encontra na pasta  `build-dev`
```
docker image build -t ex-build-dev .
```

Crie o container
```
docker container run -it -v $(pwd):/app -p 81:8000 --name python-server ex-build-dev
```

Mapeie os logs do container em um novo container com imagem debian

```
docker container run -it --volumes-from=python-server debian cat /log/http-server.log
```

#### Fazendo PUSH para o Docker Hub

Crie uma nova imagem a partir de uma tag já criada, inserindo a versão da imagem que será enviada para o Docker Hub
Como:
```
docker image tag ex-simple-build kappyh/simple-build:1.0
```

Confirme que a imagem foi criada com o comando de listagem `docker image ls`

Faça login na conta do Docker Hub(Necessário ter uma conta)
```
docker login --username=kappyh
```
Automaticamente o terminal irá solicitar a senha da conta.

Faça o push da imagem para o docker hub:
```
docker image push kappyh/simple-build:1.0
```

Acesse seu dashboard no docker hub, a imagem estará lá no fim do processo.
