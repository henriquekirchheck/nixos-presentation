---
theme: apple-basic
title: "NixOS: O sistema declarativo"
drawings:
  persist: false
transition: slide-left
mdc: true
duration: 20min
layout: intro
---

# NixOS

O Sistema Declarativo

<div class="absolute bottom-10">
  <span class="font-700">
    Henrique Kirch Heck
  </span>
  -
  <span class="font-700">
    Guilherme Galvão
  </span>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---
layout: two-cols
layoutClass: gap-16
---

# Conteudo da Apresentação

::right::

<Toc text-sm minDepth="1" maxDepth="2" />

---
transition: fade-out
---

# O que é o NixOS?

<img src="https://wiki.nixos.org/w/images/a/a1/Nix-snowflake-colours.svg" alt="NixOS" class="w-[100px] absolute top-1 right-1"/>
NixOS é um sistema operacional Linux baseado no gerenciador de pacotes Nix criado por Eelco Dolstra e Armijn Hemel e teve sua primeira versão lançada em 2003.

Ele suporta algumas funções incomuns, como:

- Gerenciamento de Configuração do Sistema;
- Empacotamento declarativo funcional;
- Cache binario;
- Upgrades Atomicos;
- Rollbacks.

Devido a sua natureza customizavel o NixOS não possui requerimentos minimos, rodando em qualquer sistema que suporta Linux. Ele tambem por padrão não faz nenhuma modificação para o Kernel Linux, então coisas como gerenciamento de memoria, scheduling, sistemas de arquivos e drivers funcionam do mesmo jeito que outros sistemas.

---
transition: slide-up
level: 2
---

# Configuração Declarativa

Uma das funções que definem o NixOS é a seu modelo de configuração declarativa. Nesse modelo, todo o sistema (incluindo pacotes instalados, serviços e configurações) é declarado em arquivos de configuração na linguagem Nix.

Mudanças para essa configuração são aplicadas de forma atomica pelo comando `nixos-rebuild switch`, que garante que você possa fazer rollback para uma configuração anterior. Muitos usuarios gerenciam suas configurações com um sistema de versionamento como o Git, fazendo a instalação de novos sistemas facil e consistente.

Diferente de distribuições tradicionais, onde a configuração do sistema é espalhado entre varios arquivos que devem ser manualmente editados, o NixOS integra o gerenciamento dessas configurações diretamente no sistema operacional. Isso faz do NixOS extramente util para deploys automatizados e reproduziveis.

---

## Exemplo de Configuração

Esse é um exemplo de um arquivo `configuration.nix`, aqui você pode instalar pacotes e configurar serviços de forma declarativa.

<v-click>Até grandes mudanças como trocar o kernel sendo usado pode ser feito em poucas linhas.</v-click>

````md magic-move
```nix
{ config, pkgs, ... }: {

  imports = [ ./hardware-configuration.nix ];

  environment.systemPackages = with pkgs; [ git ];

  services.openssh = {
    enable = true;
    openFirewall = true;
    settings = {
      PermitRootLogin = "no";
      PasswordAuthentication = false;
      KbdInteractiveAuthentication = false;
    };
  };
  services.fail2ban.enable = true;
}
```
```nix
{ config, pkgs, ... }: {

  imports = [ ./hardware-configuration.nix ];

  environment.systemPackages = with pkgs; [ git ];

  services.openssh = {
    enable = true;
    openFirewall = true;
    settings = {
      PermitRootLogin = "no";
      PasswordAuthentication = false;
      KbdInteractiveAuthentication = false;
    };
  };
  services.fail2ban.enable = true;

  boot.kernelPackages = pkgs.linuxPackages_latest;
}
```
```nix
{ config, pkgs, ... }: {

  imports = [ ./hardware-configuration.nix ];

  environment.systemPackages = with pkgs; [ git ];

  services.openssh = {
    enable = true;
    openFirewall = true;
    settings = {
      PermitRootLogin = "no";
      PasswordAuthentication = false;
      KbdInteractiveAuthentication = false;
    };
  };
  services.fail2ban.enable = true;

  boot.kernelPackages = pkgs.linuxPackages_zenmod_latest;
}
```
```nix
{ config, pkgs, ... }: {

  imports = [ ./hardware-configuration.nix ];

  environment.systemPackages = with pkgs; [ git ];

  services.openssh = {
    enable = true;
    openFirewall = true;
    settings = {
      PermitRootLogin = "no";
      PasswordAuthentication = false;
      KbdInteractiveAuthentication = false;
    };
  };
  services.fail2ban.enable = true;

  boot.kernelPackages = pkgs.linuxPackages_hardened;
}
```
````

---
level: 1
---

# O que é uma derivação?

Derivações são a parte central da linguagem Nix e do gerenciador de pacotes Nix:

- A linguagem Nix é usada para descrever derivações.
- Nix roda a derivação para criar um resultado.
- Resultados podem dai ser usados como entradas para outras derivações.

A forma primitiva de declarar uma derivação no Nix é usando a função `derivation`, porem existem abstrações como `stdenv.mkDerivation` que faz o processo de criar derivações muito mais consistentes.

---
level: 2
---

## Exemplo de derivação

Como exemplo, iremos empacotar o programa `gnu-hello`, o "Hello, World!" do projeto GNU.

````md magic-move
```nix
{ lib, stdenv, fetchurl }:

{}
```
```nix
{ lib, stdenv, fetchurl }:

stdenv.mkDerivation rec {
  pname = "hello";
  version = "2.12";
}
```
```nix
{ lib, stdenv, fetchurl }:

stdenv.mkDerivation rec {
  pname = "hello";
  version = "2.12";

  src = fetchurl {
    url = "mirror://gnu/${pname}/${pname}-${version}.tar.gz";
    sha256 = "1ayhp9v4m4rdhjmnl2bq3cibrbqqkgjbl3s7yk2nhlh8vj3ay16g";
  };
}
```
```nix
{ lib, stdenv, fetchurl }:

stdenv.mkDerivation rec {
  pname = "hello";
  version = "2.12";

  src = fetchurl {
    url = "mirror://gnu/${pname}/${pname}-${version}.tar.gz";
    sha256 = "1ayhp9v4m4rdhjmnl2bq3cibrbqqkgjbl3s7yk2nhlh8vj3ay16g";
  };

  meta = with lib; {
    description = "A familiar gretting from the GNU Project";
    license = licenses.gpl3Plus;
  };
}
```
````

---
level: 1
---

# Nix Store

Pacotes gerenciados pelo Nix são colocados na Nix Store, uma pasta imutavel normalmente localizada em `/nix/store`. Todo pacote é dado um caminho unico especificado por um hash cryptografico, o nome do pacote e a versão nesse formato `nawl092prjblbhvv16kxxbk6j9gkgcqm-git-2.14.1`.

Esses prefixos fazem hash de todos as entradas do processo de contrução, incluindo arquivos do codigo fonte, toda a arvore de dependencias, flags de compição. Isso permite que o Nix instale de forma simultanea diferentes versões de um pacote, ou até diferentes variações da mesma versão com um compilador diferente (por exemplo). Quando você adiciona, remove ou atualiza um pacote, nada é removido da Nix Store, apenas links simbolicos são adicionados, removidos ou modificados no sistema.

```sh
# Um exemplo de 3 versões do firefox instaladas simultaneamente.
/nix/store/6cw9r84aq8lvzjjwpaf2cgs6k9g1yq8a-firefox-144.0.2
/nix/store/bjzdlbmm3b302zd3hjb5864gdra605ra-firefox-141.0.3
/nix/store/zkf2l1ma5rs049i9kmcn16wp6v76xhm9-firefox-140.0.4
```

## Perfis

Para contruir um ambiente coerente, o Nix faz links simbolicos da Nix store para dentro de `profiles`. Esse é o mecanismo que permite rollbacks, já que a Nix Store é imutavel e versões antigas de perfis são mantidos, voltar para uma configuração anterior é tão facil quando mudar o link simbolico para um perfil anterior.

---

# nixpkgs

O grande diferencial do Nix como gerenciador de pacotes é a `nixpkgs`, um gigantesco repositorio de derivações mantido pela comunidade do Nix. O repositorio é mantido no GitHub e tem suporte oficial da NixOS Foundation.

O repositorio é aberto para contruibução, o que fez dele um dos maiores repositorios de software Linux.

O `nixpkgs` contem uma organização baseada em channel branches. Diferente da branch `main`, onde mudanças são aceitadas sem muito testes, channel branches são apenas atualizados depois de muitos testes serem feitos para garantir a estabilidade do sistema. Essas branches só avançam quando todos os builds e testes para um commit são completados com sucesso no sistema de build `Hydra`, que é mantido pelo NixOS com doações.

Existem varios tipos de channel branches, cada um para seu uso, mas eles podem ser categorizados em:

- Stable Channels: por exemplo `nixos-25.04`, a versão estavel mais nova do NixOS.
- Unstable Channels: `nixos-unstable` e `nixpkgs-unstable`, a versão rolling-release instavel do NixOS.

Quando definindo um sistema Nix, você pode escolher o channel que mais atende suas necessidades.

---

# Fontes

- https://wiki.nixos.org/wiki/NixOS
- https://wiki.nixos.org/wiki/Nix_(package_manager)
- https://wiki.nixos.org/wiki/Nixpkgs