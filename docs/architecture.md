# Architecture

## Overview

The Ansible Infrastructure Lab uses one centralized Ansible Control Node to configure and maintain two Ubuntu web servers.

## Infrastructure

```text
                    Ansible Control Node
                            |
                            | SSH
                            |
                 +----------+----------+
                 |                     |
                 v                     v
               web01                 web02
           192.168.1.20          192.168.1.30
                 |                     |
              Ubuntu                 Ubuntu
                 |                     |
               Nginx                 Nginx