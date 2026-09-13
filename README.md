# Linux Lab Bazzite: ambiente de desenvolvimento com Distrobox, Ubuntu e Podman

Documentação técnica da construção e validação de um ambiente de desenvolvimento no **Bazzite** usando **Distrobox**, **Ubuntu 26.04 LTS** e **Podman rootless**, mantendo os projetos armazenados fora do filesystem interno do container.

Repositório oficial: https://github.com/franklinbsa/linux-lab-bazzite-dev

## Sobre o Linux Lab

O **Linux Lab** documenta, em etapas, a construção e a validação de ambientes de desenvolvimento no Linux.

Neste repositório, o sistema host utilizado é o **Bazzite**.

A arquitetura inicial é:

```text
Bazzite
   ↓
Podman rootless
   ↓
Distrobox
   ↓
Ubuntu 26.04 LTS
   ↓
ubuntu-dev
   ↓
StorageSSD/DEV/projetos
```

## Documentação

### Camada 1 — Fundação do ambiente

A primeira camada registra a base do ambiente de desenvolvimento, incluindo:

- Bazzite como sistema host;
- Podman em modo rootless;
- Distrobox como camada de integração;
- Ubuntu 26.04 LTS no container `ubuntu-dev`;
- HOME dedicado ao ambiente de desenvolvimento;
- acesso ao `StorageSSD`;
- validação de leitura e escrita em `StorageSSD/DEV/projetos`.

➡️ [Ler a documentação completa da Camada 1](https://github.com/franklinbsa/linux-lab-bazzite-dev/blob/main/docs/01-bazzite-distrobox-ubuntu-dev.md)

## Status do projeto

A **Camada 1** está documentada e registra o estado validado do ambiente até o momento.

As próximas camadas serão adicionadas progressivamente conforme forem configuradas e validadas.

## Próximas etapas

Entre as etapas previstas para continuidade do Linux Lab estão:

- ferramentas básicas de desenvolvimento;
- Git e configuração de identidade;
- GitHub CLI e autenticação;
- Node.js e npm;
- React;
- Vite;
- Tailwind CSS;
- integração do VS Code ao container;
- extensões e fluxo de desenvolvimento;
- Codex CLI;
- publicação de aplicações;
- Cloudflare Pages;
- domínio e automação de reconstrução do ambiente.

Itens futuros serão documentados apenas depois de configurados ou validados.

## Organização do repositório

```text
linux-lab-bazzite-dev/
├── README.md
├── LICENSE
└── docs/
    └── 01-bazzite-distrobox-ubuntu-dev.md
```

Novos documentos serão adicionados à pasta `docs/` conforme a evolução do projeto.

## Objetivo

O objetivo deste projeto é registrar a construção de um ambiente de desenvolvimento Linux de forma progressiva, separando:

- o sistema operacional host;
- o ambiente de desenvolvimento;
- os arquivos reais dos projetos.

Essa separação reduz a dependência dos projetos em relação ao filesystem interno do container, mas não substitui uma estratégia de backup.

## Escopo atual

Este repositório documenta um ambiente específico.

Comandos, caminhos, versões, permissões e configurações podem variar de acordo com:

- distribuição Linux;
- hardware;
- versão das ferramentas;
- sistema de arquivos;
- ponto de montagem;
- permissões;
- contexto de uso.

Sempre revise os comandos antes de aplicá-los em outro ambiente.

## Tecnologias e conceitos abordados

- Bazzite
- Linux
- Fedora Atomic
- Podman
- Podman rootless
- Distrobox
- Ubuntu 26.04 LTS
- containers
- ambiente de desenvolvimento
- armazenamento de projetos em SSD separado

## Licença

A documentação deste projeto está licenciada sob a **Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)**.

Consulte o arquivo [LICENSE](https://github.com/franklinbsa/linux-lab-bazzite-dev/blob/main/LICENSE) para os termos completos.

Copyright © 2026 Franklin Barbosa.
