+++
title = "O que existe por trás de um jogo?"
weight = 100
bg = "bg-gradient-h"
+++

<!--: .wrap -->

## **O que existe por trás de um jogo?**

---


<!--: .wrap .size-70 ..aligncenter -->


## **Agenda**

<!--: .flexblock features  -->
- <div><h2>{{< svg fa-cubes >}}Micro serviços</h2>Separando jogos em serviços.</div>
- <div><h2>{{< svg fa-linux >}}Infraestrutura</h2>Cloud, contêineres e outros conceitos.</div>
- <div><h2>{{< svg fa-sitemap >}}Orquestração</h2>Gerenciando muitos contêineres.</div>
- <div><h2>{{< svg fa-diamond >}}Partes essenciais</h2>Pitaya, Maestro e Matchmaker.</div>
- <div><h2>{{< svg fa-line-chart >}}Problemas e soluções</h2>Alguns desafios interessantes.</div>
- <div><h2>{{< svg fa-magic >}}Data, Backend e SRE</h2>Alguns times no backstage.</div>

---

<!--: .wrap -->
<div><h2>{{< svg fa-cubes >}}Micro serviços</h2>Separando jogos em serviços.</div>

---

<!-- : .wrap -->

|||v

### **Criando um jogo no modo roots**

Todas funcionalidades do jogo são implementadas no mesmo projeto, por exemplo, os sistemas de clãs, ranking e chat. Chamamos esse tipo de arquitetura de monolito.

|||

![](images/00_mono.png)

---

<!-- : .wrap -->

|||v

### **Muitos jogos, muitos problemas**

Apesar das vantagens de ser uma arquitetura mais simples, os monolitos tornam-se o epicentro de grandes problemas com o crescimento das organizações e de seus produtos.

|||

![](/images/00_monos.png)

---

<!-- : .wrap -->

|||v

### **Serviços especializados**

As funcionalidades em comum com demais jogos são identificadas e separadas em novos projetos, tornando-se serviços que podem ser utilizados por todos os jogos, sem que estes tenham que refazer toda a lógica.

|||

![](/images/00_micros.png)

---

<!--: .wrap -->

## **Como integrar tudo isso?**
Vamos falar de infraestrutura.

--- 

<!--: .wrap -->
<div><h2>{{< svg fa-linux >}}Infraestrutura</h2>Cloud, contêineres e outros conceitos.</div>

---
<!-- : .wrap .size-70 .aligncenter -->

## O que é cloud computing?

Cloud computing ou computação em nuvem é a entrega da computação como um serviço ao invés de um produto, onde recursos compartilhados, software e informações são fornecidas, permitindo o acesso através de qualquer computador, tablet ou celular conectado à Internet.


---

<!-- : .wrap .size-70 .aligncenter -->

## O que é uma plataforma de cloud?

São recursos e serviços oferecidos por empresas especializadas em computação em nuvem. Esses produtos vão de de tecnologias de infraestrutura, como computação, armazenamento e bancos de dados, a tecnologias emergentes como machine learning e Internet das Coisas. 

![](/images/00_cloudc.png)

---

<!-- : .wrap -->

|||v

### **Onde ficam os servidores de jogos?**

Em máquinas virtuais fornecidas por plataformas como Amazon Web Services e Google Cloud Platform.

|||

![](/images/00_wheregamesrun.png)

---

<!-- : .wrap -->

|||v

### **Como distribuir tantos serviços?**

Existem várias maneiras de se empacotar e distribuir softwares. Uma das mais utilizadas para serviços são os **contêineres Linux**.

Um container pode ser entendido como um conjunto de processos que são isolados do resto do sistema. Todos arquivos necessários para executar esse sistema isolado são definidos por uma imagem, garantindo portabilidade e consistência.

|||

![](/images/00_container.png)

---

<!-- : .wrap -->

|||v

### **Rodando jogos na nuvem**

Após fazer o upload das nossas imagens para algum registry, podemos baixá-las e executá-las em máquinas virtuais nas plataformas de nuvem. Assim, através de pontos de acesso públicos, os usuários podem acessar nossos serviços através da Internet em seus dispositivos pessoais.

|||

![](/images/00_containers.png)

---

<!-- : .wrap -->

|||v

### **Muitos contêineres, muitos problemas**

Como gerenciar centenas de projetos rodando em milhares de contêineres?

|||

![](/images/00_hosts.png)

---

<!--: .wrap -->
<div><h2>{{< svg fa-sitemap >}}Orquestração</h2>Gerenciando muitos containers.</div>

---

<!-- : .wrap -->

|||v

### **Orquestradores de contêineres**

As ferramentas de orquestração de contêineres também são aplicações em nuvem que facilitam o gerenciamento de múltiplos contêineres. Seus principais objetivos são:

<!--: .description  -->
- 1. Gerenciar o ciclo de vida dos contêineres de forma autônoma;
- 2. Gerenciar volumes e rede.

|||

![](/images/00_orchestrator.png)

---

<!-- : .wrap -->

|||v

![](/images/00_orchestrators.png)

|||

### **Kubernetes**

Também conhecido como K8s, é um sistema de código aberto para automatizar a implantação, escalonamento e gerenciamento de aplicações em contêineres.

---

<!--: .wrap -->
<div><h2>{{< svg fa-diamond >}}Partes essenciais</h2>Pitaya e Maestro.</div>

---

<!-- : .wrap -->

|||v

![](/images/00_pitaya.png)

|||

### **Pitaya**

Pitaya é um framework para contrução de servidor de jogos, simples, rápido, leve e com bibliotecas de cliente para iOS, Android, Unity e outros.

---

<!-- : .wrap -->

|||v

![](/images/00_maestro.png)

|||

### **Maestro**

Maestro é um sistema utilizado para controlar as salas de jogos no Kubernetes. Funciona mantendo um conjunto de salas de jogo, aumentando ou diminuindo sua escala com base nas regras fornecidas pelo desenvolvedor.

---

<!--: .wrap -->
<div><h2>{{< svg fa-line-chart >}}Problemas e soluções</h2>Alguns desafios interessantes.</div>

---

<!-- : .wrap .size-70 .aligncenter -->

<h2>{{< svg fa-globe >}}</h2>
## Latência

Como reduzir a latência em jogos com escalas mundiais?

---

<!-- : .wrap -->

|||v

### **Kubernetes pelo mundo**

Para reduzir a latência média dos usuários, existem vários clusters Kubernetes com suas respectivas salas de jogos que atendem os usuários mais próximos.

Essa malha global de serviços é interligada por meio de tecnologias de rede modernas, que permitem com que essas regiões conversem entre si.

|||

![](/images/00_globe.png)

---

<!--: .wrap -->
<div><h2>{{< svg fa-magic >}}Data, Backend e SRE</h2>Alguns times no backstage.</div>

---

<!-- : .wrap .size-70 .aligncenter -->

## Obrigado

luan.guimaraes@wildlifestudios.com
