# Linux Lab Bazzite: ambiente de desenvolvimento com Distrobox, Ubuntu e Podman

Guia prático para construir um ambiente de desenvolvimento no **Bazzite** usando **Distrobox**, **Ubuntu 26.04 LTS** e **Podman rootless**, mantendo os projetos armazenados fora do filesystem interno do container.

## Sobre o Linux Lab

O **Linux Lab** documenta, em etapas, a construção e validação de um ambiente de desenvolvimento no Linux.

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
