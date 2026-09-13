# Bazzite com Distrobox e Ubuntu 26.04: ambiente de desenvolvimento com Podman

Guia prático para criar um ambiente de desenvolvimento no Bazzite com Distrobox, Ubuntu 26.04 LTS, Podman rootless e projetos armazenados em SSD separado.

## Linux Lab — Camada 1

Este documento registra a fundação do ambiente de desenvolvimento do Linux Lab.

A arquitetura utiliza:

- Bazzite como sistema host;
- Podman em modo rootless;
- Distrobox como camada de integração;
- Ubuntu 26.04 LTS em um container dedicado;
- HOME específico para o ambiente de desenvolvimento;
- projetos armazenados fora do filesystem interno do container;
- SSD separado como área física dos projetos.

O ambiente de desenvolvimento criado nesta camada recebeu o nome:

```text
ubuntu-dev
