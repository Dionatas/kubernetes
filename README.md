# Kubernetes 
Orquestrador de Container

## Sumário

<!-- TOC -->
- [O quê preciso saber antes de começar?](#o-quê-preciso-saber-antes-de-começar)
  - [Qual distro Linux devo usar?](#qual-distro-linux-devo-usar)
  - [Alguns sites que devemos visitar](#alguns-sites-que-devemos-visitar)
  - [O que é k8s?](#o-que-é-k8s)
  - [Arquitetura do k8s](#Arquitetura-do-k8s)
  - [Portas que devemos nos preocupar](#Portas-que-devemos-nos-preocupar)
  - [Tá, mas qual tipo de aplicação eu devo rodar sobre o k8s?](#Tá-mas-qual-tipo-de-aplicação-eu-devo-rodar-sobre-o-k8s?)
  - [Conceitos importantes do k8s](#Conceitos-importantes-do-k8s)
- [Instalação do Kubernetes](#instalação-do-kubernetes)
  - [Instalação utilizando RHEL/CentOS](#instalação-utilizando-RHEL/CentOS)
  - [Instalação utilizando Debian/Ubuntu](#instalação-utilizando-Debian/Ubuntu)
- [Instalação do Dashboard](#instalação-do-Dashboard)
- [Instalação do Metallb](#instalação-do-Metallb)
<!-- TOC -->


# O quê preciso saber antes de começar?

## Qual distro Linux devo usar?

Devido ao fato de algumas ferramentas importantes, como o ``systemd`` e ``journald``, terem se tornado padrão na maioria das principais distribuições disponíveis hoje, você não deve encontrar problemas para seguir o treinamento, caso você opte por uma delas, como Ubuntu, Debian, Red Hat, CentOS e afins.

## Alguns sites que devemos visitar

- [https://kubernetes.io/pt/](https://kubernetes.io/pt/)

- [https://github.com/kubernetes/kubernetes/](https://github.com/kubernetes/kubernetes/)

- [https://github.com/kubernetes/kubernetes/issues](https://github.com/kubernetes/kubernetes/issues)

- [https://www.cncf.io/certification/cka/](https://www.cncf.io/certification/cka/)

- [https://www.cncf.io/certification/ckad/](https://www.cncf.io/certification/ckad/)

- [https://12factor.net/pt_br/](https://12factor.net/pt_br/)

## O que é k8s?

O projeto Kubernetes foi desenvolvido pela Google, em meados de 2014, para atuar como um orquestrador de *containers* para a empresa. O Kubernetes (k8s), cujo termo em Grego significa "timoneiro", é um projeto *opensource* que conta com *design* e desenvolvimento baseados no projeto Borg, que também é da Google. Alguns outros produtos disponíveis no mercado, tais como o Apache Mesos e o Cloud Foundry, também surgiram a partir do projeto Borg.

Como Kubernetes é uma palavra difícil de se pronunciar - e de se escrever - a comunidade simplesmente o apelidou de **k8s**, seguindo o padrão [i18n](http://www.i18nguy.com/origini18n.html) (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se simplesmente "kates".

## Arquitetura do k8s

Assim como os demais orquestradores disponíveis, o k8s também segue um modelo *master/slave*, constituindo assim um *cluster*, onde para seu funcionamento devem existir no mínimo três nós: o nó *master*, responsável por padrão apenas pelo gerenciamento do *cluster*, e os demais como *workers*, executores das aplicações que nós queremos executar sobre esse *cluster*.

Embora exista a exigência de no mínimo três nós para a execução do k8s em um ambiente padrão, existem soluções para se executar o k8s em um único nó. Alguns exemplos são:

- [Minikube](https://github.com/kubernetes/minikube): Muito utilizado para implementar um *cluster* Kubernetes localmente para fins didáticos, de desenvolvimento e testes. O Minikube não deve ser utilizado para produção;

- [MicroK8S](https://microk8s.io): Desenvolvido pela [Canonical](https://canonical.com), mesma empresa que desenvolve o [Ubuntu](https://ubuntu.com). O MicroK8S pode ser utilizado em diversas distribuições e tem como público alvo desenvolvedores e profissionais de DevOps. Além disso, essa ferramenta pode ser utilizada para ambientes de produção, em especial para *Edge Computing* e IoT;

- [k3s](https://k3s.io): Desenvolvido pela [Rancher Labs](https://rancher.com), é um concorrente direto do MicroK8s, podendo ser executado inclusive em Raspberry Pi.

A figura a seguir mostra a arquitetura interna de componentes do k8s.

| ![Arquitetura Kubernetes](https://upload.wikimedia.org/wikipedia/commons/b/be/Kubernetes.png) |
|:---------------------------------------------------------------------------------------------:|
| *Arquitetura Kubernetes*                                                                      |

- **API Server**: É um dos principais componentes do k8s. Este componente fornece uma API que utiliza JSON sobre HTTP para comunicação, onde para isto é utilizado principalmente o utilitário ```kubectl```, por parte dos administradores, para a comunicação com os demais nós, como mostrado no gráfico. Estas comunicações entre componentes são estabelecidas através de requisições [REST](https://restfulapi.net);

- **etcd**: O etcd é um *datastore* chave-valor distribuído que o k8s utiliza para armazenar as especificações, status e configurações do *cluster*. Todos os dados armazenados dentro do etcd são manipulados apenas através da API. Por questões de segurança, o etcd é por padrão executado apenas em nós classificados como *master* no *cluster* k8s, mas também podem ser executados em *clusters* externos, específicos para o etcd, por exemplo.  ;

- **Scheduler**: O *scheduler* é responsável por selecionar o nó que irá hospedar um determinado *pod* (a menor unidade de um *cluster* k8s - não se preocupe sobre isso por enquanto, nós falaremos mais sobre isso mais tarde) para ser executado. Esta seleção é feita baseando-se na quantidade de recursos disponíveis em cada nó, como também no estado de cada um dos nós do *cluster*, garantindo assim que os recursos sejam bem distribuídos. Além disso, a seleção dos nós, na qual um ou mais pods serão executados, também pode levar em consideração políticas definidas pelo usuário, tais como afinidade, localização dos dados a serem lidos pelas aplicações, etc;

- **Controller Manager**: É o *controller manager* quem garante que o *cluster* esteja no último estado definido no etcd. Por exemplo: se no etcd um *deploy* está configurado para possuir dez réplicas de um *pod*, é o *controller manager* quem irá verificar se o estado atual do *cluster* corresponde a este estado e, em caso negativo, procurará conciliar ambos;

- **Kubelet**: O *kubelet* pode ser visto como o agente do k8s que é executado nos nós workers. Em cada nó worker deverá existir um agente Kubelet em execução. O Kubelet é responsável por de fato gerenciar os *pods*, que foram direcionados pelo *controller* do *cluster*, dentro dos nós, de forma que para isto o Kubelet pode iniciar, parar, e manter os *containers* e os pods em funcionamento de acordo com o instruído pelo controlador do cluster;

- **Kube-proxy**: Age como um *proxy* e um *load balancer*. Este componente é responsável por efetuar roteamento de requisições para os *pods* corretos, como também por cuidar da parte de rede do nó;

- **Container Runtime**: O *container runtime* é o ambiente de execução de *containers* necessário para o funcionamento do k8s. Em 2016 suporte ao [rkt](https://coreos.com/rkt/) foi adicionado, porém desde o início o Docker já é funcional.

## Portas que devemos nos preocupar

**MASTER**

- API Server: 6443 TCP

- etcd: 2379-2380 TCP

- Kubelet: 10250 TCP, 10255 TCP

- Scheduler: 10251 TCP

- Controller Manager: 10252 TCP

- NodePort Services: 30000-32767 TCP

**WORKERS**

- Kubelet: 10250 TCP, 12255 TCP

- NodePort Services: 30000-32767 TCP

Caso você opte pelo [Weave](https://weave.works) como *pod network*, devem ser liberadas também as portas 6783 e 6784 TCP.

## Tá, mas qual tipo de aplicação eu devo rodar sobre o k8s?

O melhor *app* para rodar em container, principalmente no k8s, são aplicações que seguem o [The Twelve-Factor App](https://12factor.net/pt_br/).

## Conceitos importantes do k8s

É importante saber que a forma como o k8s gerencia *containers* é ligeiramente diferente de outros orquestradores, como o Docker Swarm, sobretudo devido ao fato de que ele não trata os *containers* diretamente, mas sim através de *pods*. Vamos conhecer alguns dos principais conceitos que envolvem o k8s a seguir:

- **Pod**: O *pod* é o menor objeto do k8s. Como dito anteriormente, o k8s não trabalha com os *containers* diretamente, mas organiza-os dentro de *pods*, que são abstrações que dividem os mesmos recursos, como endereços, volumes, ciclos de CPU e memória. Um *pod*, embora não seja comum, pode possuir vários *containers*;

- **Controller**: Um *controller* é o objeto responsável por interagir com o *API Server* e orquestrar algum outro objeto. Exemplos de objetos desta classe são *Deployments* e *Replication Controllers*;

- **ReplicaSets**: Um *ReplicaSet* é um objeto responsável por garantir a quantidade de *pods* em execução no nó;

- **Deployment**: É um dos principais *controllers* utilizados. O *Deployment*, em conjunto com o *ReplicaSet*, garante que determinado número de réplicas de um *pod* esteja em execução nos nós *workers* do *cluster*. Além disso, o *Deployment* também é responsável por gerenciar o ciclo de vida das aplicações, onde características associadas a aplicação, tais como imagem, porta, volumes e variáveis de ambiente, podem ser especificados em arquivos do tipo *yaml* ou *json* para posteriormente serem passados como parâmetro para o *kubectl* executar o *deployment*. Esta ação pode ser executada tanto para criação quanto para atualização e remoção do *deployment*;

- **Jobs e CronJobs**: Responsáveis pelo gerenciamento de tarefas isoladas ou recorrentes.