# Apache Hadoop
## Descrição de Recursos
### Autor: Pedro Mathias Nakibar

  [Apache Mesos](https://github.com/mesosphere/mesos) é um gerenciador de cluster que oferece isolamento e compartilhamento eficiente de recursos entre aplicações ou frameworks distribuídas. Mesos é um software de código aberto originalmente desenvolvido na Universidade da Califórnia em Berkeley. Ele fica entre a camada de aplicação e o sistema operacional, tornando mais fácil a implantação e gerenciamento de aplicações em ambientes de cluster em grande escala de forma mais eficiente. Ele pode executar várias aplicações em um pool de nós compartilhados dinamicamente. Usuários proeminentes de Mesos incluem **Twitter, Airbnb, MediaCrossing, Xogito e Categorize**.
  Mesos aproveita características de kernels modernos ("cgroups" do Linux, "zones" do Solaris) para fornecer isolamento para CPU, memória, I / O, sistema de arquivos, rack de localização, etc. A grande ideia é fazer uma vasta coleção de recursos heterogêneos. Mesos introduz um mecanismo distribuído de escalonamento de dois níveis chamado ofertas de recursos. Mesos decide a quantidade de recursos que oferecerá a cada framework, enquanto os frameworks decidem quais recursos aceitar e que cálculos executar neles. É uma camada fina de compartilhamento de recursos que permite o compartilhamento de granulação fina entre diversos frameworks de computação em cluster, dando aos frameworks uma interface comum para acessar recursos em cluster. A ideia é implantar vários sistemas distribuídos a um pool compartilhado de nós, a fim de aumentar a utilização dos recursos. Várias cargas de trabalho e frameworks modernos podem ser executados no Mesos, incluindo Hadoop, Memecached, Ruby on Rails, Storm, JBoss Data Grid, MPI, Spark e Node.js, bem como vários servidores web, bancos de dados e servidores de aplicação.  

![Abstração de nó no *Apache Mesos*](http://image.slidesharecdn.com/introduction-140714121343-phpapp01/95/introduction-to-apache-mesos-12-638.jpg)

  Da mesma forma que um sistema operacional de um PC gerencia o acesso aos recursos em um computador desktop, Mesos garante que aplicações tenham acesso aos recursos de que precisam em um cluster. Ao invés de criar inúmeros clusters de servidores para diferentes partes de uma aplicação, Mesos permite compartilhar um pool de servidores que podem executar diferentes partes de sua aplicação sem que elas interfiram umas com as outras e com a capacidade de alocar dinamicamente os recursos de todo o cluster conforme necessário. Isso significa é possível facilmente transferir os recursos de um framework1 (que esteja fazendo uma análise de big-data, por exemplo) e alocá-los para framework2 (por exemplo, um servidor web), se houver muito tráfego de rede. Ele também reduz muito as etapas manuais de implantação de aplicações e pode alterar as cargas de trabalho automaticamente para fornecer tolerância a falhas e manter altas as taxas de utilização.

![Compartilhamento de recursos em todo *cluster* aumenta o rendimento e a utilização](http://image.slidesharecdn.com/txlf2014-mesosattwitter-140615135358-phpapp02/95/apache-mesos-at-twitter-texas-linuxfest-2014-16-638.jpg)

  Mesos é essencialmente o kernel de um data center, o que significa que é este o software que realmente isola as cargas de trabalho em execução uma das outras. Ele ainda precisa de ferramentas adicionais para permitir que engenheiros possam fazer suas cargas de trabalho rodarem no sistema e para gerenciar quando esses trabalhos realmente serão executados. Caso contrário, algumas cargas de trabalho pode consumir todos os recursos, ou cargas de trabalho importantes pode ser impedidas de executar por cargas de trabalho menos importantes que, por acaso, exigem mais recursos. Portanto, Mesos precisa de mais do que apenas um kernel, como o Chronos: uma substituição cron para iniciar e parar serviços automaticamente (e tratamento de falhas) que roda em cima do Mesos. A outra parte do Mesos é **Marathon**, que fornece API para iniciar, parar e escalar serviços (e o **Chronos** poderia ser um desses serviços).

![Cargas de trabalho no Chronos e no Marathon](https://lh4.googleusercontent.com/tV8rBT8WY_oESkU3W1y8VPg_03W7ZfbEm8TL9XQ3vn4zGytB5H5xAD93Cs-vu0HhEwBWE5EBpYYJZaZWV1pUXU9zdIa25q36pNZnvUiBCObe1XiO1ZwtdlU)


#### Arquitetura
  Mesos consiste em um processo mestre que gerencia daemons escravos rodando em cada nó do cluster, e frameworks que executam tarefas nestes escravos. O mestre implementa compartilhamento de granulação fina entre frameworks usando ofertas de recursos. Cada oferta de recursos é uma lista de recursos livres em vários escravos. O mestre decide quantos recursos oferecer a cada framework de acordo com uma política organizacional, tais como a repartição justa ou prioridade. Para apoiar um conjunto diversificado de políticas inter-framework de alocação, Mesos permite às organizações definir suas próprias políticas através de um módulo de alocação.
![Arquitetura Mesos com dois frameworks em execução](http://github.com/pnakibar/gerenciodocs/pocs/mesos2.png)
[Fonte](https://drive.google.com/drive/u/1/folders/0B5Xg5eCunYAmTTVnZmppcDhnazg)
  Cada framework em execução no Mesos consiste em dois componentes: um escalonador que se registra com o mestre para que possa receber ofertas de recursos, e um processo executor que é iniciado em nós escravos para executar as tarefas dos frameworks. Enquanto o mestre determina quantos recursos oferecer a cada framework, o escalonador dos frameworks seleciona quais dos recursos oferecidos usar. Quando um framework aceita os recursos oferecidos, ele passa ao Mesos uma descrição das tarefas que ele deseja executar sobre eles.
![Escalonamento de frameworks no Mesos](http://github.com/pnakibar/gerenciodocs/pocs/mesos3.png)
[Fonte](https://drive.google.com/drive/u/1/folders/0B5Xg5eCunYAmTTVnZmppcDhnazg)

  A figura acima mostra um exemplo de como um framework é escalonado para executar tarefas. Na etapa um, o escravo 1 se reporta ao mestre informando que possui 4 CPUs e 4 GB de memória livre. O mestre, em seguida, invoca o módulo de alocação, que diz a ele que todos os recursos disponíveis devem ser oferecidos ao framework 1. Na etapa dois, o mestre envia uma oferta de recursos que descreve esses recursos para framework 1. Na etapa três, o escalonador do framework responde ao mestre com informações sobre duas tarefas para serem executadas no slave, usando 2 CPUs; 1 GB de RAM para a primeira tarefa, e 1 CPUs; 2 GB de RAM para a segunda tarefa. Finalmente, na etapa quatro, o mestre envia as tarefas para o escravo, que aloca os recursos apropriados para executor do framework, que por sua vez inicia as duas tarefas (representado com bordas pontilhadas). Porque 1 CPU e 1 GB de RAM ainda estão livres, o módulo de alocação pode agora oferecê-los ao framework 2. Além disso, este processo de oferta de recursos se repete quando as tarefas terminarem e novos recursos forem liberados.
  Embora a interface fina fornecida pelo Mesos lhe permite escalar e permite que os frameworks evoluam de forma independente, um framework irá rejeitar as ofertas que não satisfaçam as suas restrições e aceitar as que satisfaçam. Em particular, descobrimos que uma política simples chamado atraso de programação, em que frameworks aguardam por um tempo limitado para adquirir os nós que armazenam os dados de entrada, produz localidade de dados quase ideal.

#### Recursos Mesos
* Mestre replicado tolerantes a falhas usando ZooKeeper
* Escalabilidade para milhares de nós
* Isolamento entre as tarefas com recipientes Linux
* Multi-escalonamento de recursos (memória e CPU)
* APIs em Java, Python e C ++ para o desenvolvimento de novas aplicações paralelas
* Web UI para visualização do estado do cluster

Há uma série de projetos de software construídas em cima de Apache Mesos:

##### Serviços de longa duração
* Aurora é um escalonador de serviços que roda em cima de Mesos, permitindo que você execute serviços de longa execução que tiram proveito da escalabilidade tolerância a falhas e isolamento de recursos do Mesos.
* Marathon é uma PaaS privada construídas no Mesos. Ele lida automaticamente com falhas de hardware ou software e garante que uma aplicação esteja sempre rodando.
* Singularity é um escalonador (API HTTP e interface web) para a execução de tarefas Mesos: processos de execução longa, tarefas pontuais e trabalhos agendados.
* SSSP é uma simples aplicação web que fornece um white-label "Megaupload" para armazenar e compartilhar arquivos no S3.

##### Processamento de Big Data
* Cray Chapel é uma produtiva linguagem de programação em paralelo. O escalonador do Chapel Mesos permite que você execute programas Chapel no Mesos.
* Dpark é um clone em Python do Spark, um framework MapReduce escrito em Python rodando no Mesos.
* Exelixi é uma framework distribuída para a execução de algoritmos genéticos em grande escala.
* Hadoop: Executando Hadoop no Mesos distribui tarefas MapReduce de forma eficiente em um cluster inteiro.
* Hama é um Framework de computação distribuída com base em técnicas de computação paralela síncronos em massa para enormes cálculos científicos, por exemplo, matriz, gráficos e algoritmos de rede.
* IPM é um sistema de transmissão de mensagens destinadas a funcionar em uma grande variedade de computadores paralelos.
* Spark é um sistema de computação de cluster rápido e de propósito geral que faz trabalhos paralelos fáceis de escrever.
* Storm é um sistema computação distribuída em tempo real. Storm faz com que seja fácil processar de forma confiável streams ilimitados de dados, fazendo para processamento em tempo real o que Hadoop fez para processamento em lote.

##### Escalonamento em Batch
* Chronos é um escalonador de tarefas distribuídas que suporta topologias de trabalho complexas. Ele pode ser usado como um substituto mais tolerante a falhas para o cron.
* Jenkins é um servidor de integração contínua. O plugin Mesos-Jenkins lhe permite iniciar dinamicamente os trabalhadores em um cluster de Mesos, dependendo da carga de trabalho.
* JobServer é um escalonador de tarefas distribuídas e processador que permite aos desenvolvedores criar tasklets de processamento em lote personalizadas usando interface Web de apontar e clicar.
* Torque é um gerenciador de recursos distribuídos que fornece controle sobre tarefas em lote e nós de computação distribuída.
Armazenamento de dados
* Cassandra é um banco de dados distribuído altamente disponível. Escalabilidade linear e tolerância a falhas comprovada em hardware commodity ou infraestrutura de nuvem tornam a plataforma perfeita para dados de missão crítica.
* ElasticSearch é um motor de pesquisa distribuída. Mesos torna fácil de executar e de escalar.
* Hypertable é um sistema de armazenamento e processamento distribuído, de alto desempenho e escalável para dados estruturados e não estruturados.
  
