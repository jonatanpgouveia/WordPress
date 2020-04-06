# WordPress with Deploy Drone CI in GitHub

[![N|Solid](https://linuxsolutions.xyz/linuxSolution.jpeg)](https://linuxsolutions.xyz)

[![Build Status](https://droneci-github.linuxsolutions.xyz/api/badges/jonatanpgouveia/WordPress/status.svg)](https://droneci-github.linuxsolutions.xyz/jonatanpgouveia/WordPress)

Projeto para WordPress de CI/CD usando [DroneCI](https://drone-github.linuxsolutions.xyz/).

## Resumo

- Todos os arquivos do WordPress ficaram ignorados
- Plugins e Temas públicos ficaram ignorados
- Plugins e Temas pagos por enquanto ficaram dentro do git
- O tema e os plugins onde o time trabalha ficara dentro do git

## Usando esse projeto como base

Depois de baixado o projeto, é necessário fazer algumas alterações.

1. Adicionar o WordPress, plugins e temas no composer, para isso use o [wpackagist](https://wpackagist.org/).

```json
{
  "require": {
    "roots/wordpress": "*",
    "wpackagist-plugin/wordpress-seo": "12.9.1"
  },
  "repositories": [
    {
      "type": "composer",
      "url": "https://wpackagist.org"
    }
  ]
}
```

2. Adicionar no .gitignore o tema usado pelo o time.

```ini
# npm e composer
wp-content/themes/{Nome do Tema}/vendor/ # <-  para ingorar a vendor do tema
wp-content/themes/{Nome do Tema}/dist/ # <- para ignorar a dist do tema

# files to keep
!.gitignore
!.gitlab-ci.yml
!README.md
!composer.json
!composer.lock
!wp-content/themes/{Nome do Tema} # <- para não ignorar o tema
```

3. Adicionar no .drone.yml o nome do tema.

```yml
kind: pipeline
type: docker
name: WordPress

steps:
- name: Install
  image: composer:1.10.1
  commands:
    - composer install
  when:
    branch:
    - master
    - homolog

- name: DeployHomolog
  image: alpine:latest
  environment:
    KEY:
      from_secret: homolog_key # Configure the secret key in the project.
    PROJECT: name-project
    HOST: name-host
    USER_HOST: root
  commands:
    - apk update && apk upgrade && apk add --no-cache rsync openssh-client
    - mkdir -p ~/.ssh
    - echo -e "$KEY" > ~/.ssh/id_rsa
    - chmod 600  ~/.ssh/id_rsa
    - 'echo -e "Host *\n\tStrictHostKeyChecking no\n\n" > ~/.ssh/config'
    - rsync -ahrvz --delete-excluded --chmod="D755,F644" --chown=www-data:www-data -e "ssh -oStrictHostKeyChecking=no " --rsync-path="sudo rsync" * $USER_HOST@$HOST:/var/www/$PROJECT.linuxsolutions.xyz/htdocs
  when:
    branch:
      - homolog
```

4. Remover os arquivos dos git.

```
 git rm -r --cache .
```