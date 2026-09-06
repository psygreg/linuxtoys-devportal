# Listas de Repositórios

As listas de repositórios permitem que desenvolvedores distribuam aplicações através do LinuxToys sem precisar criar um script shell completo para o LinuxToys.

Uma entrada de lista de repositório descreve a aplicação, de onde o LinuxToys deve obtê-la, quais sistemas são suportados, quais dependências são necessárias e qualquer configuração opcional que deva ser realizada após a instalação.

O LinuxToys converte uma entrada válida em um script temporário de instalação e o executa através do fluxo normal de execução e transações do LinuxToys.

## Localização dos arquivos

As entradas de repositório podem ser armazenadas em:

```text
scripts/repos.json
```

ou em qualquer arquivo `.json` localizado recursivamente dentro de:

```text
scripts/lists/
```

Por exemplo:

```text
scripts/
├── repos.json
└── lists/
    ├── openlogi.json
    ├── amethyst/
    │   ├── app.json
    │   ├── icon.svg
    │   └── setup.sh
    └── utilities/
        └── example.json
```

O `repos.json` é carregado primeiro por compatibilidade retroativa. Em seguida, os arquivos dentro de `scripts/lists/` são carregados recursivamente em ordem alfabética determinística.

Um arquivo JSON pode conter uma única entrada:

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "An example application.",
  "category": "utilities"
}
```

ou uma lista contendo várias entradas:

```json
[
  {
    "name": "example-one",
    "repo": "developer/example-one",
    "description": "First example.",
    "category": "utilities"
  },
  {
    "name": "example-two",
    "repo": "developer/example-two",
    "description": "Second example.",
    "category": "utilities"
  }
]
```

Arquivos JSON inválidos e entradas inválidas são ignorados individualmente, sem impedir o carregamento das demais listas de repositórios.

## Campos obrigatórios

Toda entrada deve conter estes quatro campos de texto não vazios:

| Campo         | Finalidade                                              |
| ------------- | ------------------------------------------------------- |
| `name`        | Identidade interna da aplicação no LinuxToys.           |
| `repo`        | Repositório upstream ou identificador do projeto.       |
| `description` | Descrição padrão apresentada ao usuário.                |
| `category`    | Categoria do LinuxToys em que a aplicação será exibida. |

Exemplo:

```json
{
  "name": "example-app",
  "repo": "developer/example-app",
  "description": "A useful example application.",
  "category": "utilities"
}
```

O `name` deve ser único entre todas as listas de repositórios. Os nomes são comparados sem diferenciar maiúsculas e minúsculas. Se duas entradas usarem o mesmo nome, apenas a primeira carregada será utilizada.

O `category` corresponde ao nome do diretório de categoria do LinuxToys. Por exemplo:

```json
"category": "gaming"
```

faz com que a entrada apareça na categoria `gaming` quando essa categoria for exibida.

## Campos opcionais básicos

Metadados opcionais comuns podem ser adicionados junto aos campos obrigatórios:

```json
{
  "name": "example-app",
  "repo": "developer/example-app",
  "description": "A useful example application.",
  "description_tag": "example_app_desc",
  "category": "utilities",
  "icon": "example.svg"
}
```

### `description_tag`

```json
"description_tag": "example_app_desc"
```

Se a tabela de traduções fornecida contiver essa chave, o LinuxToys usará o valor traduzido no lugar de `description`.

O campo `description` normal continua sendo obrigatório e funciona como fallback.

### `icon`

Se omitido, o LinuxToys utiliza:

```text
application-x-executable
```

Existem duas formas de fornecer um ícone.

Um nome de ícone comum ou nome de arquivo pode continuar usando a resolução padrão de ícones do LinuxToys:

```json
"icon": "example.svg"
```

ou:

```json
"icon": "application-x-executable"
```

Aplicações cujo JSON esteja dentro de `scripts/lists/` também podem incluir seu ícone junto ao arquivo da lista:

```text
scripts/lists/example/
├── app.json
└── icon.svg
```

usando:

```json
"icon": "./icon.svg"
```

Subdiretórios também são suportados:

```json
"icon": "assets/icon.png"
```

Ícones locais de listas de repositórios devem:

* usar um caminho relativo;
* permanecer dentro de `scripts/lists/`;
* existir no sistema de arquivos;
* ser SVG ou PNG.

Caso essas condições não sejam atendidas, o LinuxToys volta para `application-x-executable`.

---

## Tipos de instalação

O campo `type` informa ao LinuxToys como a aplicação deve ser instalada.

Se `type` for omitido, o valor padrão é:

```json
"type": "git"
```

Os tipos atualmente utilizáveis são:

| Tipo      | Mecanismo de instalação                             |
| --------- | --------------------------------------------------- |
| `git`     | Última versão upstream através de `pkg_fromrelease` |
| `flathub` | Aplicação Flatpak através de `pkg_flat`             |
| `native`  | Pacote da distribuição através de `pkg_install`     |
| `url`     | URL direta de pacote através de `pkg_fromurl`       |

`repository` é reservado pelo parser, mas a instalação através de repositórios de terceiros ainda não foi implementada. Entradas que utilizem esse tipo são atualmente rejeitadas e não são exibidas.

Também é possível associar tipos a determinados valores de `os`. Por exemplo, se você quiser usar um pacote `native` para o **Arch Linux** e seus derivados, com `git` para outros sistemas:

```json
"type": {
  "arch": "native",
  "all": "git"
}
```

### `git`

Essa é a opção padrão e mais simples.

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "Example application.",
  "category": "utilities",
  "type": "git"
}
```

O LinuxToys gera:

```bash
pkg_fromrelease developer/example
```

Portanto, o valor de `repo` é o valor esperado pelo helper `pkg_fromrelease` do LinuxToys.

Como `git` é o padrão, isto é equivalente:

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "Example application.",
  "category": "utilities"
}
```

### `flathub`

Use `flathub` quando a aplicação deve ser instalada como Flatpak:

```json
{
  "name": "example",
  "repo": "https://github.com/developer/example",
  "description": "Example application.",
  "category": "utilities",
  "type": "flathub",
  "package-name": "com.example.Application"
}
```

O LinuxToys executa:

```bash
pkg_flat com.example.Application
```

### Vários Flatpaks

`package-name` também pode ser uma lista:

```json
"package-name": [
  "com.example.Application",
  "com.example.Extension"
]
```

O LinuxToys instala todos os pacotes listados.

Instalações Flatpak exigem implicitamente um sistema compatível com systemd e não podem ser realizadas dentro de um container.

### `native`

Use `native` quando a aplicação já estiver disponível através do gerenciador de pacotes da distribuição:

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "Example application.",
  "category": "utilities",
  "type": "native",
  "package-name": "example"
}
```

O LinuxToys executa:

```bash
pkg_install example
```

### Vários pacotes nativos

Uma aplicação pode exigir vários pacotes:

```json
"package-name": [
  "example",
  "example-data",
  "example-plugins"
]
```

Cada pacote é instalado através de `pkg_install`.

### Nomes de pacote diferentes entre distribuições

`package-name` também pode ser um objeto:

```json
"package-name": {
  "debian": "example",
  "fedora": "example-app",
  "arch": "example-git"
}
```

Listas também podem ser utilizadas dentro do mapeamento:

```json
"package-name": {
  "debian": [
    "example",
    "example-data"
  ],
  "fedora": [
    "example-app",
    "example-assets"
  ]
}
```

Um fallback genérico pode ser fornecido através de `all`:

```json
"package-name": {
  "all": "example",
  "fedora": "example-app"
}
```

Quando o sistema atual corresponde a uma entrada específica de distribuição, essa entrada tem precedência sobre `all`.

Isso significa que uma instalação no Fedora utiliza:

```text
example-app
```

enquanto outra distribuição suportada sem um mapeamento mais específico utiliza como fallback:

```text
example
```

### Prioridade de mapeamento de pacotes nativos

Alguns sistemas expõem mais de uma chave de compatibilidade. O LinuxToys resolve os mapeamentos de pacotes usando a seguinte prioridade:

```text
ublue
deepin
zorin
pika
manjaro
cachy
ostree
ubuntu
debian
fedora
rhel
suse
solus
arch
all
```

Isso permite que distribuições derivadas sobrescrevam o comportamento da distribuição-base.

Por exemplo:

```json
"package-name": {
  "all": "example",
  "arch": "example",
  "cachy": "example-cachyos"
}
```

utiliza `example-cachyos` no CachyOS em vez do pacote genérico do Arch.

### `url`

O tipo `url` é destinado a desenvolvedores ou empresas que distribuem pacotes diretamente, como através de seu próprio CDN ou servidor de downloads.

Exemplo:

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "Example application.",
  "category": "utilities",
  "type": "url",
  "urls": {
    "deb": "https://downloads.example.org/example-amd64.deb",
    "rpm": "https://downloads.example.org/example-x86_64.rpm",
    "pkg.tar.zst": "https://downloads.example.org/example-x86_64.pkg.tar.zst",
    "appimage": "https://downloads.example.org/Example.AppImage"
  }
}
```

As chaves de URL suportadas são:

```text
deb
rpm
pacman
pkg.tar.zst
flatpak
appimage
```

As URLs devem utilizar HTTP ou HTTPS.

O LinuxToys seleciona um pacote apropriado para a distribuição atual e passa sua URL para:

```bash
pkg_fromurl URL
```

### Seleção de pacotes

O LinuxToys dá preferência a um pacote nativo sempre que houver um disponível.

Sistemas da família Debian verificam:

```text
deb
```

Isso inclui Debian, Ubuntu, Deepin, Zorin OS e PikaOS.

Sistemas da família RPM verificam:

```text
rpm
```

Isso inclui Fedora, RHEL, openSUSE, sistemas rpm-ostree e Universal Blue.

Sistemas da família Arch verificam:

```text
pkg.tar.zst
pacman
```

nessa ordem.

Isso inclui Arch Linux, CachyOS e Manjaro.

Se nenhum pacote nativo utilizável estiver disponível, o LinuxToys verifica formatos portáveis nesta ordem:

```text
appimage
flatpak
```

Por exemplo:

```json
"urls": {
  "deb": "https://example.org/app.deb",
  "rpm": "https://example.org/app.rpm",
  "appimage": "https://example.org/App.AppImage"
}
```

instalará o DEB em sistemas da família Debian, o RPM em sistemas da família RPM e poderá usar o AppImage como fallback nos demais.

Uma entrada do tipo URL só é exibida quando o LinuxToys consegue resolver uma das URLs fornecidas para o sistema atual.

---

## Compatibilidade

Entradas de listas de repositórios podem ser limitadas a determinados sistemas operacionais, ambientes de desktop, hardware, sistemas de init ou ambientes containerizados.

Campos omitidos geralmente são tratados como irrestritos.

### Sistemas operacionais

Use `os` para restringir uma aplicação a um ou mais sistemas operacionais suportados.

Um sistema:

```json
"os": "fedora"
```

Vários:

```json
"os": [
  "fedora",
  "arch",
  "debian"
]
```

Valores suportados:

```text
debian
ubuntu
cachy
arch
fedora
rhel
suse
ostree
ublue
zorin
solus
pika
deepin
manjaro
```

A entrada fica disponível quando pelo menos uma das chaves de compatibilidade de sistema solicitadas corresponde ao host.

Por exemplo:

```json
"os": [
  "fedora",
  "rhel"
]
```

permite a entrada tanto em sistemas compatíveis com Fedora quanto com RHEL.

### Ambiente de desktop

O campo opcional `desktop` restringe uma aplicação a determinados ambientes de desktop.

Valores suportados:

```text
gnome
plasma
other
```

Um desktop:

```json
"desktop": "gnome"
```

Vários:

```json
"desktop": [
  "gnome",
  "plasma"
]
```

Uma entrada é aceita quando pelo menos um dos desktops especificados corresponde ao ambiente atual.

Use esse campo apenas quando a aplicação ou integração realmente depender de um ambiente de desktop específico.

### Hardware

A compatibilidade de hardware é declarada em `hardware`.

Por exemplo:

```json
"hardware": {
  "gpu": "nvidia"
}
```

ou:

```json
"hardware": {
  "gpu": [
    "amd",
    "intel"
  ],
  "cpu": "amd"
}
```

O LinuxToys converte esses valores para suas chaves normais de compatibilidade.

Por exemplo:

```text
gpu: "amd"    -> gpu-amd
cpu: "intel"  -> cpu-intel
```

Valores que já possuam prefixo também podem ser utilizados:

```json
"hardware": {
  "gpu": "gpu-xe"
}
```

Os valores podem ser strings ou listas.

Dentro de cada classe de hardware, vários valores funcionam como alternativas. Por exemplo:

```json
"hardware": {
  "gpu": [
    "amd",
    "nvidia"
  ]
}
```

significa AMD **ou** NVIDIA, e não que ambas sejam obrigatórias.

O valor especial:

```text
all
```

não adiciona nenhum requisito de hardware.

Os nomes de hardware correspondem, em última instância, às chaves de compatibilidade expostas pelo LinuxToys. Portanto, utilize chaves suportadas pelo subsistema de compatibilidade do LinuxToys.

### systemd

O campo opcional `systemd` pode restringir explicitamente uma entrada com base no sistema de init.

Requer systemd:

```json
"systemd": "yes"
```

Requer um sistema sem systemd:

```json
"systemd": "no"
```

Omita o campo, use `null` ou uma string vazia quando ambos forem aceitáveis.

Instalações Flatpak exigem implicitamente systemd, independentemente de esse campo ser fornecido.

Declarar `services` também exige implicitamente systemd.

### Containers

A compatibilidade com containers é controlada por:

```json
"container": "allow"
```

ou:

```json
"container": "deny"
```

O padrão é:

```json
"container": "allow"
```

portanto, a maioria das entradas não precisa especificar esse campo.

Use:

```json
"container": "deny"
```

quando uma aplicação não puder ser instalada corretamente a partir de dentro de um container.

### Restrições automáticas para containers

O LinuxToys rejeita automaticamente entradas dentro de containers quando a instalação selecionada instalaria:

```text
Flatpak
AppImage
```

Isso se aplica mesmo que:

```json
"container": "allow"
```

tenha sido especificado explicitamente.

A mesma restrição se aplica quando uma entrada declara uma dependência do Flathub.

Essa proteção existe porque instalações Flatpak e AppImage não devem ser aninhadas dentro do fluxo de containers suportado.

---

## Dependências

O campo opcional `dependencies` instala pacotes antes da aplicação principal.

As dependências são uma lista de objetos.

Os tipos de dependência atualmente suportados são:

```text
native
flathub
```

### Dependência nativa

```json
"dependencies": [
  {
    "type": "native",
    "package-name": "git"
  }
]
```

Dependências nativas suportam exatamente as mesmas formas de `package-name` que uma aplicação nativa.

Um único pacote:

```json
{
  "type": "native",
  "package-name": "git"
}
```

Vários pacotes:

```json
{
  "type": "native",
  "package-name": [
    "git",
    "curl"
  ]
}
```

Pacotes específicos por distribuição:

```json
{
  "type": "native",
  "package-name": {
    "debian": "libexample-dev",
    "fedora": "example-devel",
    "arch": "example"
  }
}
```

E os próprios mapeamentos podem conter listas de pacotes:

```json
{
  "type": "native",
  "package-name": {
    "debian": [
      "libexample1",
      "libexample2"
    ],
    "fedora": [
      "example-libs",
      "example-data"
    ]
  }
}
```

Se nenhum mapeamento de dependência nativa puder ser resolvido para o host, a própria aplicação é considerada incompatível e não é exibida.

### Dependência do Flathub

```json
"dependencies": [
  {
    "type": "flathub",
    "package-name": "org.example.Runtime"
  }
]
```

Vários Flatpaks são permitidos:

```json
{
  "type": "flathub",
  "package-name": [
    "org.example.Runtime",
    "org.example.Extension"
  ]
}
```

Uma dependência do Flathub exige implicitamente systemd e torna a entrada incompatível com instalação dentro de containers.

### Várias dependências

Diferentes dependências podem ser combinadas:

```json
"dependencies": [
  {
    "type": "native",
    "package-name": {
      "debian": "example-helper",
      "fedora": "example-helper",
      "arch": "example-helper"
    }
  },
  {
    "type": "flathub",
    "package-name": "org.example.Runtime"
  }
]
```

As dependências são instaladas antes da aplicação principal.

---

## Overrides

Comportamentos adicionais de instalação podem ser declarados utilizando:

```json
"overrides": {}
```

As chaves de override atualmente suportadas são:

```text
flatpak
pre
post
```

Outras chaves fazem com que a entrada seja rejeitada.

### Hook de pré-instalação

`pre` é executado antes das dependências e dos comandos de instalação da aplicação.

Para operações curtas, ele pode conter shell inline:

```json
"overrides": {
  "pre": "mkdir -p \"$HOME/.config/example\""
}
```

Como esse conteúdo é inserido diretamente no script Bash de instalação gerado, ele deve ser mantido pequeno e previsível.

### Hook de pós-instalação

`post` é executado depois das dependências, da instalação da aplicação, dos overrides de Flatpak e da configuração de serviços.

Exemplo:

```json
"overrides": {
  "post": "touch \"$HOME/.config/example/installed\""
}
```

### Scripts externos de pre/post

Configurações mais complexas podem ser distribuídas como um script separado dentro de `scripts/lists/`.

Exemplo de estrutura:

```text
scripts/lists/example/
├── app.json
├── pre-install.sh
└── post-install.sh
```

O JSON pode referenciá-los através de:

```json
"overrides": {
  "pre": {
    "script": "example/pre-install.sh"
  },
  "post": {
    "script": "example/post-install.sh"
  }
}
```

Esses hooks são executados através do helper `run_list_hook` do LinuxToys.

Os caminhos dos hooks devem ser relativos. Caminhos absolutos e caminhos que tentem sair de `scripts/lists/` são rejeitados.

Por exemplo, isto é inválido:

```json
"pre": {
  "script": "../outside.sh"
}
```

Scripts externos são mais apropriados quando a configuração é complexa demais para ser expressa razoavelmente como um pequeno comando inline.

## Ordem de execução

O procedimento de instalação gerado é executado nesta ordem:

```text
hook pre

dependências

instalação da aplicação principal

overrides de Flatpak

serviços systemd

hook post

mensagem de sucesso
```

Essa ordem é importante ao escrever hooks. Um hook `pre` não pode assumir que a aplicação já foi instalada, enquanto um hook `post` pode.

---

## Overrides de Flatpak

O LinuxToys pode aplicar permissões de Flatpak após a instalação utilizando seu helper `flatpak_override`.

Exemplo:

```json
"overrides": {
  "flatpak": [
    {
      "scope": "user",
      "type": "filesystem",
      "setting": "xdg-config/example",
      "target": "com.example.Application"
    }
  ]
}
```

Todo override de Flatpak exige quatro campos:

| Campo     | Descrição                                      |
| --------- | ---------------------------------------------- |
| `scope`   | `user` ou `system`                             |
| `type`    | Tipo de override do Flatpak                    |
| `setting` | Permissão ou configuração passada ao LinuxToys |
| `target`  | Aplicação Flatpak de destino                   |

Tipos de override suportados:

```text
fs
name
dbus
share
env
runtime
device
socket
filesystem
talk-name
talk-dbus
```

Vários overrides podem ser aplicados:

```json
"overrides": {
  "flatpak": [
    {
      "scope": "user",
      "type": "filesystem",
      "setting": "xdg-config/example",
      "target": "com.example.Application"
    },
    {
      "scope": "user",
      "type": "device",
      "setting": "dri",
      "target": "com.example.Application"
    }
  ]
}
```

O LinuxToys traduz cada entrada em uma chamada equivalente a:

```bash
flatpak_override SCOPE TYPE SETTING TARGET
```

---

## Serviços systemd

O campo `services` pode instruir o LinuxToys a habilitar e iniciar imediatamente unidades systemd após a instalação.

Qualquer entrada que declare `services` é automaticamente restrita a hosts com systemd.

### Um serviço de sistema

A forma mais curta é:

```json
"services": "example"
```

Por padrão, isso é tratado como um serviço de sistema e se torna:

```text
example.service
```

O LinuxToys efetivamente executa:

```bash
sudo systemctl enable --now example.service
```

### Vários serviços de sistema

```json
"services": [
  "example",
  "example-helper"
]
```

Ambos usam escopo de sistema por padrão.

### Serviços de sistema e de usuário

Para controle explícito:

```json
"services": {
  "system": [
    "example.service"
  ],
  "user": [
    "example-tray.service"
  ]
}
```

Serviços de sistema são habilitados com:

```bash
sudo systemctl enable --now UNIT
```

Serviços de usuário são habilitados com:

```bash
systemctl --user enable --now UNIT
```

O LinuxToys registra essas operações em seu mapa de transações para que possam participar do fluxo normal de reversão.

### Sufixos de unidades

Se nenhum sufixo reconhecido de unidade systemd for fornecido, o LinuxToys adiciona automaticamente:

```text
.service
```

Portanto:

```json
"services": "example"
```

e:

```json
"services": "example.service"
```

são equivalentes.

Os sufixos de unidade reconhecidos incluem:

```text
.service
.socket
.timer
.path
.mount
.automount
.target
.slice
.scope
.device
.swap
```

Isso também permite unidades que não sejam serviços:

```json
"services": {
  "system": [
    "example.socket",
    "example.timer"
  ]
}
```

---

## Exemplo completo

O exemplo abaixo demonstra a maior parte dos recursos atualmente suportados pelas listas de repositórios:

```json
{
  "name": "example-app",
  "repo": "https://github.com/example/example-app",
  "description": "A cross-platform example application.",
  "description_tag": "example_app_desc",
  "category": "utilities",
  "icon": "./icon.svg",

  "type": "url",

  "urls": {
    "deb": "https://downloads.example.org/example-app-amd64.deb",
    "rpm": "https://downloads.example.org/example-app-x86_64.rpm",
    "pkg.tar.zst": "https://downloads.example.org/example-app-x86_64.pkg.tar.zst",
    "appimage": "https://downloads.example.org/ExampleApp-x86_64.AppImage"
  },

  "os": [
    "debian",
    "ubuntu",
    "deepin",
    "zorin",
    "pika",
    "fedora",
    "rhel",
    "suse",
    "ostree",
    "ublue",
    "arch",
    "cachy",
    "manjaro"
  ],

  "desktop": [
    "gnome",
    "plasma"
  ],

  "hardware": {
    "gpu": [
      "amd",
      "intel",
      "nvidia"
    ]
  },

  "systemd": "yes",
  "container": "deny",

  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": [
          "curl",
          "git"
        ],
        "fedora": [
          "curl",
          "git"
        ],
        "arch": [
          "curl",
          "git"
        ],
        "all": "curl"
      }
    }
  ],

  "overrides": {
    "pre": {
      "script": "example-app/pre-install.sh"
    },

    "flatpak": [],

    "post": {
      "script": "example-app/post-install.sh"
    }
  },

  "services": {
    "system": [
      "example-app.service"
    ],
    "user": [
      "example-app-tray.service"
    ]
  }
}
```

Uma estrutura de diretórios correspondente poderia ser:

```text
scripts/lists/example-app/
├── app.json
├── icon.svg
├── pre-install.sh
└── post-install.sh
```

---
<a id="minimal-examples"></a>

## Exemplos mínimos

### Release do GitHub

```json
{
  "name": "example",
  "repo": "developer/example",
  "description": "An example application.",
  "category": "utilities"
}
```

### Flathub

```json
{
  "name": "example",
  "repo": "https://github.com/developer/example",
  "description": "An example application.",
  "category": "utilities",
  "type": "flathub",
  "package-name": "com.example.Application"
}
```

### Pacote nativo

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "An example application.",
  "category": "utilities",
  "type": "native",
  "package-name": {
    "debian": "example",
    "fedora": "example",
    "arch": "example"
  }
}
```

### URLs diretas de pacotes

```json
{
  "name": "example",
  "repo": "https://example.org",
  "description": "An example application.",
  "category": "utilities",
  "type": "url",
  "urls": {
    "deb": "https://example.org/download/example.deb",
    "rpm": "https://example.org/download/example.rpm",
    "appimage": "https://example.org/download/Example.AppImage"
  }
}
```

---

## Comportamento de validação

As entradas de listas de repositórios são validadas antes de aparecerem no LinuxToys.

Uma entrada é silenciosamente ignorada quando seus campos obrigatórios estão ausentes, seu tipo de instalação é inválido ou inutilizável, seus requisitos de compatibilidade não correspondem à máquina atual, um pacote nativo ou URL não pode ser resolvido para o host, uma dependência não pode ser satisfeita, um override está malformado, sua definição de serviços é inválida, sua configuração de container é inválida ou seu `name` duplica uma entrada carregada anteriormente.

Isso significa que desenvolvedores devem testar suas entradas de lista de repositório em todas as classes de sistema que pretendem suportar.

Uma entrada de lista de repositório é exposta ao restante do LinuxToys de forma semelhante a um script normal do LinuxToys. O LinuxToys gera uma identidade virtual no formato:

```text
repo://NAME
```

e materializa um script shell temporário apenas quando a entrada precisa ser executada.

Instalações provenientes de listas de repositórios são marcadas como reversíveis e participam do fluxo normal de instalação e transações do LinuxToys.

---

## Escolhendo um método de instalação

Use `git` quando seu projeto publica artefatos instaláveis através de releases compatíveis com o parser de releases do LinuxToys.

Use `flathub` quando Flatpak for o método de distribuição pretendido.

Use `native` quando a aplicação já estiver disponível nos repositórios normais das distribuições.

Use `url` quando você publicar pacotes diretamente e quiser que o LinuxToys selecione o formato mais adequado para cada distribuição.

Dependências, campos de compatibilidade, serviços e overrides podem então ser adicionados sobre esses tipos de instalação conforme necessário.

Mantenha as entradas o mais simples possível. As listas de repositórios são destinadas a aplicações que possam ser instaladas de forma declarativa. Se o processo de instalação de uma aplicação exigir uma quantidade significativa de lógica personalizada, um script tradicional do LinuxToys ainda pode ser a forma de integração mais apropriada.
