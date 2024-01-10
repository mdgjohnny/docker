Quando procuro entender uma nova tecnologia, gosto de começar pela seguinte questão: qual problema está sendo resolvido aqui? Perguntar-se a respeito do problema de imediato já nos traz questões relacionadas àquele contexto, isto é, quais foram as outras tentativas de dar conta daquela dificuldade, se as houver? E o que esta tecnologia faz de diferente das outras?

No caso dos contêineres, vamos primeiro partir de uma definição ingênua do problema e torná-la cada vez mais complexa, com o benefício do percurso histórico que realizaremos. Uma tentativa rudimentar de dar forma ao problema seria: como posso isolar uma aplicação de todas as outras coisas que estão acontecendo no meu sistema? No entanto, a pergunta ainda parece omitir uma informação importante: por que seria necessário isolar as coisas? Quais as dificuldades que surgem quando não há uma barreira separando o domínio de cada processo numa máquina?
## Percurso histórico

É neste ponto que devemos nos recordar de como as aplicações eram implantadas ao longo da história. De início, a infraestrutura das aplicações consistia em poucas camadas: o servidor, o sistema operacional e a aplicação. Era como se entre a aplicação e a carcaça houvesse apenas um mediador. Acontece que às vezes muitas aplicações rodavam ao mesmo tempo e consumiam muitos recursos, tornando outras aplicações ineficientes. À época, uma possível solução era rodar cada aplicação em um servidor diferente. Só que, se você pensar bem, verá que se trata de uma solução pouco eficiente, já que havia muito custo de manutenção (já que a quantidade de servidores necessários poderia aumentar indefinidamente) e pouca escalabilidade (uma vez que os recursos de cada servidor seriam subaproveitados, já que estariam destinados a apenas uma aplicação, e sempre que uma aplicação precisasse de mais recursos, seria preciso migrar para cada vez mais servidores, etc).

A solução para a implantação tradicional foi recorrer à virtualização. Em suma, múltiplas máquinas virtuais (VMs) poderiam executar no mesmo servidor físico. Não só diminuímos o custo e tornamos a solução mais escalável como também ganhamos o benefício de uma segurança mais robusta, já que a virtualização, ao impedir que uma aplicação tenha livre acesso a outra, garante uma camada adicional de segurança ao processo. As máquinas virtuais permitem uma alocação mais eficiente dos recursos do servidor físico e, com isso, tornam possível que uma aplicação seja escalável, alocando mais ou menos recursos conforme for o caso. Pense em cada máquina virtual como a cópia de uma máquina real dotada de todos os componentes necessários, até mesmo de seu próprio sistema operacional, que roda num hardware também virtualizado.

Enfim chegamos aos contêineres, que se assemelham em muito às máquinas virtuais: ambos têm um sistema próprio de arquivos, alocação determinada dos recursos da CPU, etc. A diferença principal consiste no seguinte: enquanto as máquinas virtuais incluem um sistema operacional completo para cada instância, os contêineres compartilham o mesmo sistema operacional do *host* sem deixar de isolar os processos entre si. Além disso, cada contêiner encapsula a aplicação e todas as suas dependências em uma única unidade (a imagem, como veremos mais adiante), tornando-o mais portátil e consistente do que as VMs. Enfim, a inovação dos contêineres reside na sua capacidade de proporcionar isolamento e eficiência sem a sobrecarga de um sistema operacional completo, como é o caso das máquinas virtuais.

## Um relance por trás dos bastidores

Você já entendeu o conceito de contêiner, o problema que eles solucionam e as vantagens que trouxeram em comparação aos métodos tradicionais de implantação. No entanto, talvez ainda reste a pergunta: o que rola por trás dos bastidores do contêiner? Quais são as ferramentas que ele implementa?

Quando nos referimos a um contêiner, estamos nos referindo a um processo isolado que roda numa máquina hospedeira. Este processo não se comunica com os outros processos do servidor, sejam eles quais forem. O contêiner tira partido de algumas características do Linux, como [*namespaces*](https://man7.org/linux/man-pages/man7/namespaces.7.html) e [*cgroups*](https://man7.org/linux/man-pages/man7/cgroups.7.html). Não vamos nos alongar nisso agora. O que a gente precisa saber é que ambas as características permitem circunscrever determinados recursos globais do sistema a uma camada de abstração só dele. Ou seja, é como se cada processo fosse soberano em seu próprio espaço e tivesse monopólio sobre aquele recurso do sistema. As abstrações dos *namespaces* permitem a existência simultânea de múltiplos grupos de processos, cada um com uma visão particular do sistema. É meio macabro, digamos assim, mas ajuda pensar na estrutura da conteinerização como semelhante a um panóptico. O agente que supervisiona o sistema é capaz de avistar todo o interior do edifício, ou seja, as jaulas de execução contendo seus respectivos processos, ao passo que todo processo dentro daquela jaula tem apenas uma vista parcial da totalidade do sistema, que lhe é inatingível e impenetrável. A analogia é, por definição, inexata, mas serve para nos ajudar a visualizarmos melhor o processo.

![Panóptico de Jeremy Bentham](panoptico.png)


## A imagem, ou o contêiner em ação

Vamos recapitular os seguintes fatos que aprendemos até agora sobre o contêiner:
- É uma unidade coesa composta de um amontoado de coisas (chamadas dependências) de que uma aplicação precisa para ser executada.
- Essa unidade se decompõe num grupo de processos que são alocados a determinados espaços de execução.
-  Tais processos costumam ter certas limitações, como por exemplo permissões restritas, uma capacidade limitada de alocação de memória, etc.
-  As restrições são implantadas por determinadas funcionalidades do kernel do Linux (como *cgroups*, *namespaces*, *pivot_root*, etc).
-  Pode rodar em uma máquina virtual, local ou na nuvem.
-  Está isolado de todos os outros processos do servidor hospedeiro.
-  É a instância executável de uma *imagem*.

Quando falamos em imagem, não queremos dizer nada além de um arquivo estático que contém todas as instruções executáveis para que aquele amontoado de coisas rode à perfeição. Pense numa receita de bolo. Assim como uma receita, uma imagem contém uma lista de todos os ingredientes de que um contêiner depende para ser executado: o código da aplicação, as bibliotecas do sistema, o interpretador da linguagem (digamos), a base do sistema operacional... Com a diferença de que, neste caso, o bolo é preparado automaticamente, segundo um conjunto de instruções pré-determinadas. 

Além disso, uma imagem é constituída de camadas específicas, que são compostas de diretórios. Na receita, por exemplo, teríamos uma camada para cada ferramente de que precisaríamos para fazer o bolo. No contêiner, poderíamos ter uma camada com Ubuntu, outra com alguma aplicação específica, e assim vai, todas empilhadas sobre uma camada base.

A diferença, portanto, entre uma imagem e um contêiner consiste no fato de que a imagem é a descrição do ambiente do contêiner, ao passo que o contêiner é a instância executável dessa imagem, é a imagem "ganhando vida". A imagem é imutável (as camadas que as compõe não mudam); o contêiner é mutável (as alterações em tempo de execução são armazenadas na camada de escrita, sem afetar a camada original). Contêiner e imagem são conceitos recíprocos: não posso pensar no contêiner sem pensar na imagem que ele corporifica.

## E o que o Docker tem a ver?

Docker nada mais é que uma peça central nessa arquitetura de conteinerização. Lembra da nossa imagem do panóptico? Podemos pensar no Docker como o agente que supervisiona o sistema. No entanto, a analogia se revela limitada quando lembramos que a arquitetura cliente-servidor do Docker permite que ele não só supervisione como construa, executa e orquestre esses processos. O ponto de partida de todo esse processo é algo chamado *Dockerfile*, que é um arquivo de texto com uma série de instruções para construir a imagem. Cada instrução numa *Dockerfile* corresponde mais ou menos a uma camada de uma imagem. O formato de uma *Dockerfile* seria, então:
```
# Comentário
INSTRUÇÃO argumentos
```

Em regra, todo *Dockerfile* começa com a mesma instrução: FROM. ([Há exceções](https://docs.docker.com/engine/reference/builder/#format), claro.) FROM nada mais é do que uma das instruções básicas que compõem o vocabulário do Docker.

Entre algumas das instruções elementares, podemos listar as seguintes:
- **FROM**:
	- Cria uma novo estágio de construção a partir de uma imagem base.
	- Exemplo: `FROM ubuntu:18.04` constrói a imagem a partir da imagem base do Ubuntu 18.04.
- **WORKDIR**:
	-  Define o diretório de trabalho em que as instruções serão executadas.
	-  Exemplo: `WORKDIR /app` define o diretório `/app` como o diretório de trabalho.
- **RUN**: executa comandos durante a construção (caso você precise instalar Python, por exemplo, você usaria `RUN apt-get update && apt-get install -y python3`)
- **ENTRYPOINT**:
	- Configura os comandos que serão executados durante a inicialização do contêiner. São "pontos de entrada", isto é, a maneira como a aplicação virá à luz.
	- Exemplo: `ENTRYPOINT ["python3"]` faz com que o contêiner execute o interpretador do Python.
- **EXPOSE**:
	- Descreve quais portas a aplicação deve escutar durante o tempo de execução.
	-  Exemplo: `EXPOSE 80` indica que o contêiner escutará a porta 80.

E assim por diante. Você pode conferir o restante das instruções na [documentação oficial do Docker](https://docs.docker.com/engine/reference/builder).

Vejamos agora um exemplo prático de Dockerfile:
```
FROM python:3.8 
WORKDIR /app 
COPY . /app 
RUN pip install -r requirements.txt CMD ["python", "./meu_script.py"]
```

Este Dockerfile cria uma imagem para uma aplicação Python a partir de uma imagem base do Python 3.8, configura o diretório de trabalho, copia os arquivos da aplicação, instala as dependências e define o comando padrão para executar a aplicação. E pronto.

### Depois do Dockerfile

Quando o Dockerfile estiver pronto, podemos construir a imagem com `docker build`. Há diversas nuances possíveis durante a construção de imagem, que podem ser consultadas na referência oficial do Docker.

Após a imagem estar pronta, chegamos à etapa de execução com `docker run`. Aqui, o que acontece é que o comando que você digita é enviado para o *Docker Daemon*, que é como o gerente do sistema. O *Docker Daemon* é o "cérebro" que gerencia as imagens. Ele entende o que você quer fazer e cuida de todos os detalhes técnicos para iniciar a aplicação dentro de um contêiner. Configura o ambiente isolado, aloca os recursos necessários e garante que tudo esteja funcionando conforme esperado.

Poderíamos construir e rodar a imagem anterior assim:

```
docker build -t minha_imagem_python
docker run -d -p 5000:5000 minha_imagem_python
```

Agora que a imagem `minha_imagem_python` foi construída, você pode iniciar um contêiner a partir dela. Este comando executa o contêiner em modo 'detached' (`-d`), mapeando a porta 5000 do host para a porta 5000 do contêiner.

Se quisermos verificar quais contêineres estão rodando e, posteriormente, encerrá-los, podemos utilizar o comando `docker ps`, que mostra todos os contêineres ativos do momento seguido do ID ou nome do contêiner.  Se quisermos encerrá-lo, usaríamos `docker stop`.

### Resumindo tudo

O Docker lê uma série de instruções de um arquivo de texto chamado *Dockerfile* e constrói uma imagem a partir dele. A imagem é uma espécie de "receita" para uma aplicação, contendo todas suas dependências e configurações. Quando executamos `docker run`, constrói-se o contêiner a partir dessa imagem. O contêiner é já em si uma instância em execução da imagem; é a imagem viva, que funciona em um ambiente isolado, semelhante a um sistema operacional, mas que compartilha o kernel do sistema hospedeiro.

Lembrando que este texto nasceu da perspectiva de alguém que está  está aprendendo Docker enquanto escreve. Meu propósito foi digerir e reorganizar as informações que encontrei, com o intuito de torná-las mais compreensíveis tanto para mim quanto para outros que possam estar com as mesmas dúvidas. Este é apenas o início da jornada; o que apresentei aqui é uma base teórica, um esqueleto conceitual que ajuda a entender o que acontece nos bastidores. A próxima etapa é colocar tudo isso em prática. 

Uma boa jornada a todos!

### Referências

https://docs.docker.com/engine/reference
https://docs.docker.com/guides/walkthroughs/what-is-a-container/
https://medium.com/@saschagrunert/demystifying-containers-part-i-kernel-space-2c53d6979504
https://kubernetes.io/docs/concepts/overview/#why-containers

