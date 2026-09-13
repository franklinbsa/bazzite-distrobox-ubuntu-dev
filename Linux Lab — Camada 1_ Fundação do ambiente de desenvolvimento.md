---
title: "Bazzite com Distrobox e Ubuntu: fundação de um ambiente de desenvolvimento no Linux Lab"
description: "Documentação da Camada 1 do Linux Lab com Bazzite, Podman rootless, Distrobox, Ubuntu LTS e projetos armazenados em SSD separado."
author: "Franklin Barbosa"
status: "Concluída"
tags:
  - "bazzite"
  - "distrobox"
  - "ubuntu"
  - "podman"
  - "linux"
---

# Bazzite com Distrobox e Ubuntu: fundação do ambiente de desenvolvimento do Linux Lab

> Este tutorial documenta um ambiente específico. Adapte comandos, caminhos, versões e permissões conforme sua máquina, distribuição, hardware e contexto de uso.

## Visão geral

Esta documentação registra a **Camada 1 do Linux Lab**, responsável pela fundação do ambiente de desenvolvimento.

O objetivo desta fase foi manter o **Bazzite como sistema host**, evitando transformar o sistema principal no local de instalação de toda a futura stack de desenvolvimento.

Para isso, foi adotada uma arquitetura baseada em:

- Bazzite como host;
- Podman executado em modo rootless;
- Distrobox como camada de integração;
- Ubuntu LTS dentro de um container dedicado;
- HOME específico para o ambiente de desenvolvimento;
- projetos armazenados fisicamente fora do filesystem interno do container;
- `StorageSSD` como área de armazenamento dos projetos.

O novo ambiente recebeu o nome:

```text
ubuntu-dev
```

A imagem utilizada foi:

```text
docker.io/library/ubuntu:26.04
```

O principal resultado desta camada foi validar que o container consegue acessar e escrever na área:

```text
/run/media/system/StorageSSD/DEV/projetos
```

Assim, a arquitetura separa três responsabilidades:

```text
sistema operacional
        ↓
Bazzite

ambiente de desenvolvimento
        ↓
ubuntu-dev

arquivos reais dos projetos
        ↓
StorageSSD/DEV/projetos
```

Essa separação ajuda a reduzir a dependência do filesystem interno do container. Ela não representa, por si só, uma estratégia completa de backup.

## Para quem é este tutorial

Este material pode ser útil para:

- iniciantes que estão aprendendo a diferença entre host e container;
- usuários de Bazzite;
- desenvolvedores que desejam separar ferramentas de desenvolvimento do sistema principal;
- pessoas estudando Distrobox;
- usuários de distribuições da família Fedora Atomic;
- pessoas interessadas em Podman rootless;
- usuários que desejam manter projetos fora do filesystem interno de containers;
- profissionais interessados em ambientes reproduzíveis;
- comunidade open source que queira adaptar, revisar ou melhorar esse processo.

Não é necessário dominar containers para compreender a arquitetura. Os principais conceitos são explicados ao longo do documento.

## Resultado esperado

Ao final da Camada 1, o estado validado do ambiente é:

- host principal executando Bazzite;
- KDE Plasma utilizado como ambiente gráfico;
- sessão Wayland;
- Distrobox disponível no host;
- Distrobox observado na versão `1.8.2.5`;
- Podman disponível no host;
- Podman observado na versão `5.8.4`;
- Podman operando em modo rootless;
- container preexistente `ia-nvidia` preservado;
- novo container `ubuntu-dev` criado;
- imagem do novo container definida como `docker.io/library/ubuntu:26.04`;
- Ubuntu 26.04 LTS confirmado internamente;
- HOME dedicado confirmado;
- acesso do container ao `StorageSSD`;
- acesso à área `DEV`;
- acesso à pasta `DEV/projetos`;
- escrita real em `DEV/projetos` validada;
- remoção do arquivo utilizado no teste validada.

As ferramentas de desenvolvimento propriamente ditas pertencem às próximas camadas.

## Contexto da fase

O Linux Lab é organizado em camadas para evitar misturar a preparação da infraestrutura com a instalação de ferramentas, autenticações, frameworks e publicação de aplicações.

Nesta primeira camada, o foco ficou restrito a:

```text
Bazzite
    ↓
Podman + Distrobox
    ↓
Ubuntu
    ↓
StorageSSD
```

O ambiente Bazzite registrado no projeto utiliza a imagem/base:

```text
ghcr.io/ublue-os/bazzite-nvidia-open:stable
```

O projeto também registra o uso de:

```text
Desktop: KDE Plasma
Sessão: Wayland
```

Um container anterior chamado:

```text
ia-nvidia
```

já existia e foi preservado.

A imagem associada a esse container foi registrada como:

```text
docker.io/library/ubuntu:24.04
```

A criação do `ubuntu-dev` teve uma finalidade diferente: estabelecer um ambiente dedicado ao desenvolvimento sem reutilizar ou destruir o container já existente.

## Fatos confirmados, hipóteses e pendências

### Fatos confirmados

O contexto disponível registra os seguintes estados:

| Item | Estado observado |
|---|---|
| Host | Bazzite |
| Imagem/base registrada do host | `ghcr.io/ublue-os/bazzite-nvidia-open:stable` |
| Desktop | KDE Plasma |
| Sessão | Wayland |
| Distrobox | `1.8.2.5` |
| Container engine | Podman |
| Podman | `5.8.4` |
| Podman rootless | `true` |
| Container preexistente | `ia-nvidia` |
| Imagem do `ia-nvidia` | `docker.io/library/ubuntu:24.04` |
| Novo container | `ubuntu-dev` |
| Imagem do `ubuntu-dev` | `docker.io/library/ubuntu:26.04` |
| Usuário validado no container | `franklin` |
| Ponto de acesso observado ao SSD | `/run/media/system/StorageSSD` |
| Área de desenvolvimento | `/run/media/system/StorageSSD/DEV` |
| Diretório de projetos | `/run/media/system/StorageSSD/DEV/projetos` |
| Escrita em `DEV/projetos` | Validada |
| Remoção do arquivo de teste | Validada |

A validação interna do `ubuntu-dev` registrou:

```text
PRETTY_NAME="Ubuntu 26.04.1 LTS"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
CONTAINER_ID=ubuntu-dev
Usuário: franklin
```

O HOME dedicado observado foi:

```text
/home/franklin/.local/share/distrobox-homes/ubuntu-dev
```

Também foi observado que a representação de usuário/grupo da raiz do `StorageSSD` dentro do container poderia ser diferente da esperada visualmente, mas a área `DEV` permaneceu funcional para o usuário de desenvolvimento.

O critério prático utilizado foi o funcionamento real da área necessária, e não a alteração indiscriminada das permissões de todo o SSD.

### Hipóteses úteis

As interpretações abaixo ajudam a compreender a arquitetura, mas não devem ser confundidas com novos testes realizados.

**Hipótese 1 — reconstruibilidade**

Manter os projetos fora do filesystem interno do `ubuntu-dev` reduz a dependência dos dados em relação à existência daquele container específico.

Isso favorece a ideia futura de tratar o container como ambiente reconstruível.

A automação completa dessa reconstrução ainda não pertence a esta camada.

**Hipótese 2 — HOME dedicado como separação organizacional**

O HOME específico do `ubuntu-dev` ajuda a separar configurações e dotfiles pertencentes ao ambiente de desenvolvimento das configurações principais do usuário.

Isso não transforma o Distrobox em uma máquina virtual ou sandbox de segurança completa.

**Hipótese 3 — funções distintas para SSD, Git e backup**

O `StorageSSD`, um futuro repositório Git/GitHub e uma estratégia de backup podem desempenhar funções diferentes.

Eles não devem ser tratados como equivalentes.

### Pendências

As seguintes etapas não foram concluídas nesta camada:

- [ ] validar e documentar uma eventual montagem permanente do `StorageSSD`;
- [ ] instalação definitiva de `build-essential`;
- [ ] instalação e configuração definitiva do Git;
- [ ] configuração da identidade Git;
- [ ] instalação do GitHub CLI;
- [ ] autenticação no GitHub;
- [ ] configuração de repositórios privados;
- [ ] configuração SSH para GitHub;
- [ ] instalação e validação de Node.js;
- [ ] instalação e validação de npm;
- [ ] React;
- [ ] Vite;
- [ ] Tailwind CSS;
- [ ] configuração definitiva de Python para desenvolvimento;
- [ ] integração do VS Code ao container;
- [ ] extensões do VS Code;
- [ ] Codex;
- [ ] Cloudflare Pages;
- [ ] aplicação React de validação;
- [ ] publicação de aplicações;
- [ ] configuração de domínio;
- [ ] arquivos definitivos de reconstrução;
- [ ] automação completa da recriação do ambiente.

A existência de configuração em `/etc/fstab` para esse SSD não foi confirmada.

**Informação não confirmada no contexto.**

## Arquitetura utilizada

A arquitetura validada nesta camada pode ser representada assim:

```text
Bazzite host
    │
    ├── Podman rootless
    │
    └── Distrobox
           │
           └── ubuntu-dev
                  │
                  ├── Ubuntu 26.04 LTS
                  │
                  ├── HOME dedicado
                  │
                  └── acesso aos projetos
                         │
                         ↓
StorageSSD
    │
    └── DEV
         │
         └── projetos
```

### Host

O Bazzite continua sendo o sistema operacional principal.

A proposta desta camada não foi substituir o Bazzite por Ubuntu nem criar dual boot especificamente para desenvolvimento.

### Podman

O Podman atua como container engine.

O modo validado foi:

```text
rootless: true
```

Rootless significa, de maneira simplificada, que o mecanismo de containers está sendo utilizado pelo usuário sem executar o container engine inteiro como root.

Isso não significa que todos os processos internos de um container possuam automaticamente o mesmo modelo de permissões do host.

### Distrobox

O Distrobox utiliza um container engine, neste caso Podman, para disponibilizar outra distribuição Linux de maneira fortemente integrada ao host.

Por essa característica, um Distrobox não deve ser entendido como uma máquina virtual completamente separada.

Também não deve ser tratado como uma sandbox de segurança absoluta.

O ambiente pode ter acesso a recursos do host conforme a integração oferecida e a configuração utilizada.

### Ubuntu

O ambiente selecionado para desenvolvimento recebeu o nome:

```text
ubuntu-dev
```

e utiliza a imagem:

```text
docker.io/library/ubuntu:26.04
```

A validação interna mostrou:

```text
Ubuntu 26.04.1 LTS
Resolute Raccoon
```

### Projetos

Os projetos não foram destinados ao filesystem interno do container.

O caminho validado foi:

```text
/run/media/system/StorageSSD/DEV/projetos
```

Essa é uma decisão importante da arquitetura.

O container representa o ambiente.

O SSD contém os arquivos de projeto destinados a sobreviver independentemente da estrutura interna do container.

Isso, isoladamente, não garante proteção contra falha de disco, exclusão acidental ou corrupção de dados.

## Estrutura de diretórios

Os caminhos efetivamente confirmados para esta camada são:

```text
/run/media/system/StorageSSD/
└── DEV/
    └── projetos/
```

O projeto também registra a preservação de uma área denominada:

```text
IA_LOCAL
```

e da pasta de filesystem:

```text
lost+found
```

A criação de outras subpastas dentro de `DEV` não possui comando ou saída literal recuperados no contexto utilizado para esta documentação.

**Informação não confirmada no contexto.**

Existe registro de criação de uma estrutura de backup durante a organização da fase, porém a composição completa dessa estrutura e todos os seus comandos não foram recuperados com evidência suficiente para serem reproduzidos aqui.

**Informação não confirmada no contexto.**

A eventual ocultação visual de `lost+found` no Dolphin por meio de um arquivo `.hidden` também não possui evidência operacional suficiente recuperada para ser apresentada como etapa concluída neste documento.

**Informação não confirmada no contexto.**

Por esse motivo, nenhum comando relacionado a `.hidden` é reproduzido nesta versão.

## Pré-requisitos

### Conhecimento recomendado

É útil compreender minimamente:

- como abrir um terminal;
- diferença entre arquivo e diretório;
- caminhos absolutos no Linux;
- diferença entre host e container;
- conceito básico de usuário e permissões;
- conceito de ponto de montagem;
- diferença entre armazenamento e backup.

Não é necessário dominar Podman ou Distrobox previamente.

### Ferramentas necessárias

As ferramentas confirmadas no ambiente documentado são:

- Bazzite;
- Distrobox;
- Podman;
- um SSD acessível como `StorageSSD`;
- imagem Ubuntu utilizada pelo `ubuntu-dev`.

### Ambiente confirmado

```text
Host: Bazzite
Imagem/base: ghcr.io/ublue-os/bazzite-nvidia-open:stable
Desktop: KDE Plasma
Sessão: Wayland
Usuário: franklin

Distrobox: 1.8.2.5
Container engine: Podman
Podman: 5.8.4
Podman rootless: true
```

O container preexistente que precisava ser preservado era:

```text
ia-nvidia
```

O novo ambiente desta camada é:

```text
ubuntu-dev
```

### Informações ainda pendentes

A montagem permanente do `StorageSSD` não está comprovada pela documentação recuperada.

**Informação não confirmada no contexto.**

Nenhuma configuração de `/etc/fstab` deve ser inferida a partir do caminho observado em:

```text
/run/media/system/StorageSSD
```

## O que foi feito

A sequência registrada nesta camada foi:

1. validar o ambiente Bazzite;
2. validar Distrobox;
3. validar Podman;
4. confirmar operação rootless;
5. identificar e preservar o container `ia-nvidia`;
6. organizar uma área dedicada para desenvolvimento no `StorageSSD`;
7. utilizar a estrutura `DEV/projetos`;
8. definir um HOME específico para o novo Distrobox;
9. definir o nome `ubuntu-dev`;
10. criar o novo container com Ubuntu 26.04;
11. realizar sua primeira inicialização;
12. permitir que o Distrobox efetuasse a configuração inicial do ambiente;
13. confirmar o sistema operacional dentro do container;
14. confirmar `CONTAINER_ID`;
15. confirmar usuário;
16. confirmar HOME;
17. confirmar acesso ao `StorageSSD`;
18. confirmar acesso a `DEV/projetos`;
19. executar um teste real de escrita;
20. remover com sucesso o arquivo utilizado no teste.

A documentação termina nesse estado funcional.

## Passo a passo

### Passo 1 — Validar o host

**Objetivo**

Confirmar qual sistema permaneceria responsável pelo computador antes de introduzir outro ambiente Linux.

**O que foi feito**

O sistema principal foi identificado como Bazzite.

Também foram registrados:

```text
Imagem/base: ghcr.io/ublue-os/bazzite-nvidia-open:stable
Desktop: KDE Plasma
Sessão: Wayland
```

**Comando ou configuração**

O projeto contém evidência anterior de inspeção do sistema com ferramentas de identificação do ambiente, porém o comando específico responsável por estabelecer todos os campos acima durante esta Camada 1 não foi recuperado como uma sequência única.

**Informação não confirmada no contexto.**

**Como validar**

O estado registrado para esta fase considera o Bazzite como host e não o Ubuntu.

**Cuidados ou problemas possíveis**

Não confundir o sistema host com o sistema executado dentro do Distrobox.

### Passo 2 — Validar Distrobox e Podman

**Objetivo**

Confirmar que a camada de containers necessária já estava disponível antes da criação do `ubuntu-dev`.

**O que foi feito**

Foram observados:

```text
Distrobox: 1.8.2.5
Podman: 5.8.4
Container engine: Podman
Podman rootless: true
```

**Comando ou configuração**

Os comandos literais utilizados para produzir essas quatro verificações não foram recuperados integralmente no contexto disponível.

**Informação não confirmada no contexto.**

Nenhum comando substituto é apresentado aqui para evitar transformar uma reconstrução provável em histórico executado.

**Como validar**

O resultado registrado confirmou Distrobox, Podman e operação rootless antes da criação do novo container.

**Cuidados ou problemas possíveis**

Versões futuras do Bazzite podem disponibilizar versões diferentes dessas ferramentas.

Os números acima descrevem este ambiente documentado.

### Passo 3 — Preservar o container existente

**Objetivo**

Evitar que a criação do novo ambiente de desenvolvimento interferisse no ambiente anterior.

**O que foi feito**

Foi identificado um container existente:

```text
ia-nvidia
```

utilizando:

```text
docker.io/library/ubuntu:24.04
```

Esse container foi preservado.

Um segundo container foi criado especificamente para desenvolvimento, em vez de reaproveitar o anterior.

**Comando ou configuração**

Nenhum comando de remoção ou recriação do `ia-nvidia` faz parte desta camada.

O comando exato utilizado para listar os containers durante esta validação não foi recuperado.

**Informação não confirmada no contexto.**

**Como validar**

Ao final da organização, o novo ambiente recebeu outro nome:

```text
ubuntu-dev
```

mantendo separadas as duas finalidades.

**Cuidados ou problemas possíveis**

Não remover containers existentes apenas porque outra finalidade será adicionada ao computador.

### Passo 4 — Preparar a área dos projetos no StorageSSD

**Objetivo**

Evitar que os projetos dependessem exclusivamente do filesystem interno do `ubuntu-dev`.

**O que foi feito**

Foi utilizada a área:

```text
/run/media/system/StorageSSD/DEV
```

com a pasta:

```text
/run/media/system/StorageSSD/DEV/projetos
```

`IA_LOCAL` e `lost+found` foram tratados como estruturas que não deveriam ser removidas durante a organização do disco.

**Comando ou configuração**

O comando literal utilizado para criar `DEV` e `DEV/projetos` não foi recuperado com evidência suficiente nesta documentação.

**Informação não confirmada no contexto.**

**Como validar**

Posteriormente, o `ubuntu-dev` conseguiu acessar a pasta e realizar escrita real dentro de:

```text
/run/media/system/StorageSSD/DEV/projetos
```

**Cuidados ou problemas possíveis**

O diretório:

```text
lost+found
```

não deve ser tratado como lixo apenas porque aparece na raiz de determinados filesystems Linux.

Também não há justificativa registrada para aplicar permissões globais como:

```text
chmod -R 777
```

em todo o SSD.

Esse procedimento não faz parte desta documentação.

### Passo 5 — Definir um HOME dedicado

**Objetivo**

Separar as configurações e dotfiles do futuro ambiente de desenvolvimento das configurações principais utilizadas no host.

**O que foi feito**

O HOME observado no `ubuntu-dev` foi:

```text
/home/franklin/.local/share/distrobox-homes/ubuntu-dev
```

**Comando ou configuração**

A sintaxe literal utilizada na criação do container para selecionar esse HOME não foi recuperada nos logs disponíveis para esta documentação.

**Informação não confirmada no contexto.**

**Como validar**

Dentro do ambiente criado, o HOME observado apontou para:

```text
/home/franklin/.local/share/distrobox-homes/ubuntu-dev
```

**Cuidados ou problemas possíveis**

HOME dedicado melhora a organização de configurações.

Ele não deve ser interpretado como prova de isolamento completo em relação ao host.

### Passo 6 — Criar o `ubuntu-dev`

**Objetivo**

Criar um ambiente Ubuntu separado para receber futuramente as ferramentas de desenvolvimento.

**O que foi feito**

Foram utilizados:

```text
Nome: ubuntu-dev
Imagem: docker.io/library/ubuntu:26.04
```

**Comando ou configuração**

O comando literal completo de `distrobox create` executado nesta fase não foi recuperado com segurança no material disponível.

**Informação não confirmada no contexto.**

Não será apresentada uma reconstrução provável desse comando como se fosse o comando histórico.

**Como validar**

O container passou a existir com:

```text
CONTAINER_ID=ubuntu-dev
```

**Cuidados ou problemas possíveis**

A criação do container não significa que Git, Node.js, VS Code ou qualquer outra stack de desenvolvimento já tenha sido configurada.

Essas etapas pertencem às próximas camadas.

### Passo 7 — Realizar a primeira inicialização

**Objetivo**

Permitir que o Distrobox preparasse o container para uso integrado.

**O que foi feito**

O `ubuntu-dev` foi inicializado e passou pelo processo inicial de configuração do Distrobox.

O Distrobox realiza configuração interna necessária para integrar o ambiente containerizado ao usuário e ao host.

**Comando ou configuração**

O comando literal utilizado para entrar pela primeira vez no `ubuntu-dev` não foi recuperado de forma suficiente para ser registrado como comando executado.

**Informação não confirmada no contexto.**

**Como validar**

Depois da inicialização, foi possível realizar as verificações internas descritas nos próximos passos.

**Cuidados ou problemas possíveis**

A primeira entrada em um Distrobox pode executar preparação adicional do ambiente.

Isso faz parte do funcionamento do Distrobox e não deve ser confundido com a instalação da futura stack de desenvolvimento.

### Passo 8 — Confirmar o Ubuntu dentro do container

**Objetivo**

Garantir que o ambiente criado correspondia ao Ubuntu pretendido.

**O que foi feito**

A leitura interna do sistema registrou:

```text
PRETTY_NAME="Ubuntu 26.04.1 LTS"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
```

**Comando ou configuração**

A validação utilizou informações equivalentes às disponíveis em `/etc/os-release`.

O comando literal executado para exibir essas linhas não foi recuperado no contexto disponível.

**Informação não confirmada no contexto.**

**Como validar**

O resultado observado identifica:

```text
Ubuntu 26.04.1 LTS
```

**Cuidados ou problemas possíveis**

A versão observada descreve o container desta documentação e não deve ser convertida automaticamente em requisito universal para outros usuários.

### Passo 9 — Confirmar identidade do container, usuário e HOME

**Objetivo**

Verificar se a sessão correspondia ao novo ambiente e utilizava o usuário e HOME planejados.

**O que foi feito**

Foram registrados:

```text
CONTAINER_ID=ubuntu-dev
Usuário: franklin
HOME=/home/franklin/.local/share/distrobox-homes/ubuntu-dev
```

**Comando ou configuração**

Os comandos literais utilizados para produzir todas essas verificações não foram recuperados.

**Informação não confirmada no contexto.**

**Como validar**

Os resultados observados confirmaram simultaneamente:

- identidade do container;
- usuário de trabalho;
- HOME dedicado.

**Cuidados ou problemas possíveis**

O prompt do terminal sozinho não deve ser o único critério utilizado para determinar em qual ambiente uma sessão está.

### Passo 10 — Validar acesso ao StorageSSD

**Objetivo**

Confirmar que a separação entre container e armazenamento de projetos funcionava na prática.

**O que foi feito**

O container conseguiu acessar:

```text
/run/media/system/StorageSSD
```

e a área:

```text
/run/media/system/StorageSSD/DEV
```

**Comando ou configuração**

O comando literal utilizado para listar ou inspecionar esses diretórios não foi recuperado.

**Informação não confirmada no contexto.**

**Como validar**

O teste avançou até a pasta:

```text
/run/media/system/StorageSSD/DEV/projetos
```

e posteriormente realizou escrita nela.

**Cuidados ou problemas possíveis**

O fato de o caminho estar acessível nessa sessão não comprova que a montagem do SSD foi configurada permanentemente para todos os boots.

### Passo 11 — Validar escrita em `DEV/projetos`

**Objetivo**

Verificar funcionalmente que o usuário dentro do `ubuntu-dev` conseguia trabalhar com arquivos reais no diretório destinado aos projetos.

**O que foi feito**

Foi criado um arquivo de teste dentro de:

```text
/run/media/system/StorageSSD/DEV/projetos
```

A operação de escrita foi bem-sucedida.

**Comando ou configuração**

O nome do arquivo e o comando literal utilizado para criá-lo não foram recuperados com evidência suficiente.

**Informação não confirmada no contexto.**

**Como validar**

A validação confirmou que a área `DEV/projetos` permitia escrita pelo ambiente de desenvolvimento.

Esse resultado é mais relevante para esta fase do que tentar modificar preventivamente as permissões de todo o `StorageSSD`.

**Cuidados ou problemas possíveis**

Não aplicar permissões globais ao disco apenas por causa de diferenças visuais de proprietário ou grupo apresentadas dentro de um namespace de container.

### Passo 12 — Remover o arquivo de teste

**Objetivo**

Encerrar o teste sem deixar o arquivo temporário criado apenas para validação.

**O que foi feito**

O arquivo utilizado no teste de escrita foi removido com sucesso.

**Comando ou configuração**

O nome do arquivo e o comando literal utilizado para a remoção não foram recuperados.

**Informação não confirmada no contexto.**

**Como validar**

O contexto registra a remoção bem-sucedida do arquivo de teste.

Nesse ponto, a validação funcional desta camada foi encerrada.

**Cuidados ou problemas possíveis**

Este passo não deve ser interpretado como autorização para utilizar comandos de remoção recursiva em diretórios de projetos.

## Validações realizadas

| Validação | Resultado |
|---|---|
| Host Bazzite | Confirmado |
| KDE Plasma | Confirmado |
| Wayland | Confirmado |
| Distrobox disponível | Confirmado |
| Distrobox `1.8.2.5` | Confirmado |
| Podman disponível | Confirmado |
| Podman `5.8.4` | Confirmado |
| Container engine Podman | Confirmado |
| Podman rootless | `true` |
| Existência do `ia-nvidia` | Confirmada |
| Imagem do `ia-nvidia` | `docker.io/library/ubuntu:24.04` |
| Preservação do `ia-nvidia` | Confirmada |
| Dry-run anterior à criação | **Informação não confirmada no contexto.** |
| Criação do `ubuntu-dev` | Confirmada |
| Imagem do `ubuntu-dev` | `docker.io/library/ubuntu:26.04` |
| Primeira inicialização | Confirmada |
| Configuração inicial pelo Distrobox | Confirmada |
| Ubuntu interno | `Ubuntu 26.04.1 LTS` |
| `VERSION_ID` | `26.04` |
| `CONTAINER_ID` | `ubuntu-dev` |
| Usuário | `franklin` |
| HOME dedicado | Confirmado |
| Acesso ao `StorageSSD` | Confirmado |
| Acesso ao `DEV` | Confirmado |
| Acesso ao `DEV/projetos` | Confirmado |
| Escrita em `DEV/projetos` | Confirmada |
| Remoção do arquivo de teste | Confirmada |
| Montagem permanente do SSD | **Informação não confirmada no contexto.** |

## Problemas encontrados e soluções

| Problema | Causa provável | Solução aplicada | Status |
|---|---|---|---|
| Representação de usuário/grupo na raiz do `StorageSSD` podia aparecer diferente dentro do container | Diferenças de representação entre host, container e namespaces de usuário | Não foi aplicada alteração global de permissões. A área necessária foi validada funcionalmente por acesso, escrita e remoção em `DEV/projetos` | Funcionalmente validado |

Não foram incluídos outros problemas sugeridos como possibilidade porque os logs recuperados não comprovam de forma suficiente que tenham ocorrido nesta fase.

Em especial:

- tentativa de executar diretamente o nome de um container como comando: **Informação não confirmada no contexto.**
- erro de permissão ao criar `.hidden`: **Informação não confirmada no contexto.**
- criação de `.hidden` usando privilégio administrativo: **Informação não confirmada no contexto.**

## Checklist final da Camada 1

### Fundação

- [x] Bazzite mantido como host
- [x] Distrobox validado
- [x] Podman validado
- [x] Podman rootless validado
- [x] container `ia-nvidia` preservado
- [x] área `DEV` utilizada no `StorageSSD`
- [x] pasta `DEV/projetos` disponível
- [x] HOME dedicado do ambiente de desenvolvimento validado
- [x] container `ubuntu-dev` criado
- [x] imagem `docker.io/library/ubuntu:26.04` utilizada
- [x] primeira inicialização concluída
- [x] Ubuntu interno validado
- [x] `CONTAINER_ID=ubuntu-dev` validado
- [x] usuário `franklin` validado
- [x] HOME dedicado validado
- [x] acesso ao `StorageSSD` validado
- [x] acesso a `DEV/projetos` validado
- [x] escrita em `DEV/projetos` validada
- [x] arquivo temporário do teste removido

### Ainda pendente

- [ ] validar montagem permanente do `StorageSSD`
- [ ] instalar definitivamente `build-essential`
- [ ] configurar Git
- [ ] configurar identidade Git
- [ ] instalar GitHub CLI
- [ ] autenticar no GitHub
- [ ] configurar SSH do GitHub
- [ ] definir fluxo de repositórios privados
- [ ] instalar Node.js
- [ ] instalar npm
- [ ] validar React
- [ ] validar Vite
- [ ] validar Tailwind CSS
- [ ] configurar definitivamente Python para desenvolvimento
- [ ] integrar VS Code ao `ubuntu-dev`
- [ ] configurar extensões do VS Code
- [ ] configurar Codex
- [ ] configurar Cloudflare Pages
- [ ] criar aplicação React de validação
- [ ] publicar aplicação
- [ ] configurar domínio
- [ ] finalizar arquivos de reconstrução
- [ ] automatizar recriação completa do ambiente

## Riscos e cuidados

### Distrobox não é uma máquina virtual

O Distrobox cria containers fortemente integrados ao host.

Isso significa que o ambiente não deve ser descrito como se fosse uma máquina virtual totalmente independente.

A existência de um HOME dedicado também não transforma o container em uma sandbox completa.

### Projetos não devem depender apenas do container

Nesta arquitetura, os projetos são destinados a:

```text
/run/media/system/StorageSSD/DEV/projetos
```

e não apenas ao filesystem interno do `ubuntu-dev`.

Essa separação reduz o acoplamento entre dados e ambiente.

Ainda assim, arquivos presentes somente nesse SSD continuam sujeitos aos riscos daquele dispositivo.

### SSD separado não significa backup completo

Há diferenças importantes entre quatro conceitos:

**Projeto fora do container**

O projeto continua disponível independentemente do filesystem interno do container enquanto o armazenamento externo permanecer íntegro e acessível.

**Cópia em outro dispositivo**

Cria redundância física adicional, dependendo de como a cópia for mantida.

**Git/GitHub**

Git registra histórico de versões quando os arquivos são efetivamente versionados e enviados ao repositório correspondente.

Git e GitHub ainda não foram configurados nesta camada.

**Backup**

Backup exige uma política própria de cópia, retenção e recuperação.

A existência do `StorageSSD` não comprova que exista backup completo dos projetos.

### Não alterar permissões indiscriminadamente

A diferença de proprietário ou grupo exibida dentro de um container não deve resultar automaticamente em alteração recursiva de permissões no disco inteiro.

O critério desta camada foi funcional:

```text
acessar
↓
ler
↓
escrever
↓
remover o arquivo de teste
```

Não foi registrada necessidade de aplicar permissões abertas globalmente.

### Não apagar `lost+found`

A presença de:

```text
lost+found
```

na raiz de determinados filesystems Linux não significa que a pasta seja lixo ou possa ser removida apenas para melhorar a aparência no gerenciador de arquivos.

### Preservar ambientes existentes

O `ia-nvidia` já possuía outra função e foi preservado.

Criar um ambiente novo para uma finalidade nova reduz o risco de misturar responsabilidades ou danificar configurações anteriores.

### Confirmar montagem permanente separadamente

O caminho observado nesta fase foi:

```text
/run/media/system/StorageSSD
```

Isso comprova acesso durante os testes realizados.

Não comprova configuração permanente de montagem após reinicializações.

Nenhum `/etc/fstab` é documentado como configurado nesta camada.

## SEO do tutorial

### Palavra-chave principal

```text
Bazzite Distrobox Ubuntu para desenvolvimento
```

### Palavras-chave secundárias

- Bazzite ambiente de desenvolvimento;
- Distrobox Ubuntu;
- Ubuntu no Bazzite;
- Podman rootless Bazzite;
- Distrobox para desenvolvimento;
- container Ubuntu no Linux;
- Bazzite Fedora Atomic desenvolvimento;
- HOME dedicado Distrobox;
- projetos fora do container;
- SSD para projetos Linux;
- ambiente Ubuntu no Distrobox;
- Podman rootless;
- Linux Atomic desenvolvimento.

### Intenção de busca

O tutorial atende principalmente usuários tentando responder:

> Como usar Bazzite como host e criar um ambiente Ubuntu separado com Distrobox e Podman para desenvolvimento sem armazenar os projetos apenas dentro do container?

Também ajuda a compreender dúvidas relacionadas a:

- como separar host e ambiente de desenvolvimento;
- como usar Ubuntu dentro do Bazzite;
- como organizar projetos fora de um container;
- qual o papel do Podman em um Distrobox;
- o que significa Podman rootless;
- para que serve um HOME dedicado no Distrobox.

### Título SEO sugerido

```text
Bazzite + Distrobox + Ubuntu: como criar uma base separada para desenvolvimento
```

### Meta description sugerida

```text
Como organizar Bazzite, Distrobox, Podman rootless e Ubuntu em um ambiente de desenvolvimento com projetos armazenados fora do container.
```

### Termos técnicos explicados

**Bazzite:** distribuição Linux baseada no ecossistema Fedora Atomic utilizada como sistema host neste projeto.

**Fedora Atomic:** família de sistemas Fedora baseada em uma abordagem de sistema operacional por imagem, diferente do modelo tradicional de modificar livremente todos os pacotes do sistema-base.

**Distrobox:** ferramenta que cria ambientes Linux em containers usando engines como Podman e oferece forte integração desses ambientes com o host.

**Podman:** ferramenta utilizada para criação e execução de containers. Neste projeto é o container engine utilizado pelo Distrobox.

**Rootless:** execução do mecanismo de containers pelo usuário sem depender de uma sessão do container engine executada integralmente como root.

**Container:** ambiente baseado em isolamento de processos e filesystem, compartilhando componentes do kernel do host. Não é equivalente a uma máquina virtual completa.

**HOME:** diretório principal associado ao usuário. Aplicações normalmente armazenam nele configurações, preferências e dados específicos do usuário.

**HOME dedicado:** neste projeto, um HOME separado foi utilizado para o `ubuntu-dev` com o objetivo organizacional de evitar que suas configurações e dotfiles fossem misturados diretamente ao HOME utilizado normalmente no host.

**LTS:** sigla para *Long Term Support*. Identifica versões Ubuntu com período estendido de manutenção oficial.

**Filesystem:** estrutura usada pelo sistema operacional para organizar arquivos, diretórios, metadados e permissões em um dispositivo de armazenamento.

**Ponto de montagem:** caminho no filesystem através do qual um dispositivo ou outro filesystem passa a ser acessível.

**Host:** sistema operacional principal no qual o mecanismo de containers está sendo executado.

**Imagem de container:** base utilizada para criar um container. Nesta camada, o `ubuntu-dev` foi criado a partir de:

```text
docker.io/library/ubuntu:26.04
```

### Perguntas frequentes

#### O Ubuntu substituiu o Bazzite?

Não.

O Bazzite continua sendo o host.

O Ubuntu é utilizado dentro do container `ubuntu-dev`.

#### Distrobox é uma máquina virtual?

Não.

Distrobox utiliza containers e possui forte integração com o host.

Ele não deve ser tratado como uma VM completamente independente.

#### Por que usar Ubuntu dentro do Bazzite?

Nesta arquitetura, o Ubuntu foi escolhido como ambiente separado para receber futuramente as ferramentas de desenvolvimento, mantendo essa responsabilidade fora do sistema-base do host.

Esta documentação não afirma que essa abordagem seja universalmente superior a outras.

#### Onde ficam os projetos?

O caminho validado nesta fase é:

```text
/run/media/system/StorageSSD/DEV/projetos
```

#### Os projetos ficam dentro do `ubuntu-dev`?

O diretório destinado aos projetos está fisicamente fora do filesystem interno do container e é acessado pelo ambiente `ubuntu-dev`.

#### Posso apagar o container sem perder os projetos?

A arquitetura busca desacoplar os projetos do filesystem interno do container porque eles estão armazenados no `StorageSSD`.

Isso não deve ser interpretado como autorização para remover containers sem antes verificar caminhos, volumes, dados e dependências relevantes.

#### O StorageSSD já é um backup?

Não.

Armazenar o projeto em outro SSD em relação ao filesystem interno do container não equivale, por si só, a manter uma estratégia completa de backup.

#### GitHub já protege esses projetos?

Não nesta camada.

Git, GitHub CLI, autenticação e repositórios fazem parte das próximas etapas.

#### O Podman está funcionando sem root?

O estado registrado nesta camada mostrou:

```text
Podman rootless: true
```

#### Qual Ubuntu foi validado?

O ambiente interno registrou:

```text
Ubuntu 26.04.1 LTS
VERSION_ID="26.04"
Resolute Raccoon
```

#### Por que usar um HOME dedicado?

O HOME dedicado ajuda a separar configurações e dotfiles do ambiente `ubuntu-dev` das configurações utilizadas normalmente no host.

Isso é uma decisão de organização, não uma garantia de isolamento de segurança.

#### O StorageSSD será montado automaticamente após reiniciar?

A montagem permanente não foi comprovada no contexto desta camada.

**Informação não confirmada no contexto.**

#### Foi configurado `/etc/fstab`?

**Informação não confirmada no contexto.**

Nenhuma configuração de `/etc/fstab` é documentada como concluída nesta fase.

## Links de referência

As referências abaixo são oficiais e servem para aprofundar os conceitos utilizados nesta documentação.

**Bazzite — documentação oficial**

[Bazzite Documentation](https://docs.bazzite.gg/)

A documentação oficial descreve o Bazzite como uma imagem personalizada baseada na família Fedora Atomic Desktop.

**Distrobox — documentação oficial**

[Distrobox Documentation](https://distrobox.it/)

A documentação oficial explica a integração do Distrobox com o host e seu uso com container engines.

**Distrobox — criação de containers e HOME personalizado**

[distrobox-create documentation](https://distrobox.it/usage/distrobox-create/)

A referência documenta, entre outras opções, a possibilidade de escolher um HOME personalizado para um container.

**Podman — documentação oficial**

[Podman Documentation](https://docs.podman.io/)

A documentação do Podman descreve o funcionamento de containers rootless e os conceitos associados.

**Ubuntu — ciclo oficial de versões**

[Ubuntu Release Cycle](https://ubuntu.com/about/release-cycle)

A Canonical registra oficialmente o Ubuntu 26.04 como uma versão LTS lançada em abril de 2026.

## Próxima fase sugerida

**Fase seguinte sugerida:**

**Camada 2 — Ferramentas básicas de desenvolvimento no Ubuntu**

**Objetivo:**

Preparar e validar as ferramentas fundamentais dentro do `ubuntu-dev`, sem poluir o host Bazzite.

Essa etapa deverá ser documentada separadamente e não é considerada concluída nesta Camada 1.

## Resumo para continuidade

A Camada 1 do Linux Lab foi encerrada com o Bazzite mantido como host, Podman operando em modo rootless e Distrobox utilizado para criar o ambiente `ubuntu-dev` baseado em Ubuntu 26.04. O container utiliza um HOME dedicado em `/home/franklin/.local/share/distrobox-homes/ubuntu-dev` e conseguiu acessar, escrever e remover um arquivo de teste na área de projetos `/run/media/system/StorageSSD/DEV/projetos`. O container preexistente `ia-nvidia` foi preservado. A próxima etapa é a **Camada 2 — Ferramentas básicas de desenvolvimento no Ubuntu**, sem considerar Git, Node.js, VS Code ou outras ferramentas como já configuradas.

## Créditos

Documentação criada por **Franklin Barbosa**.

Material compartilhado como colaboração para a comunidade, com o objetivo de ajudar outras pessoas a documentarem, reproduzirem e melhorarem processos técnicos de forma clara, prática e responsável.

**Franklin Barbosa — colaboração técnica e documentação para a comunidade.**
