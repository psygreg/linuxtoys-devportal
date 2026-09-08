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

<a id="git-package"></a>

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

<a id="flathub-package"></a>

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

<a id="native-package"></a>

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

<a id="url-fetching"></a>

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

<a id="single-binary"></a>

## Aplicativos de Binário Único

O LinuxToys pode instalar aplicativos distribuídos como um **único binário executável**, sem exigir um pacote nativo, AppImage, Flatpak ou tarball.

Isso é útil para aplicativos cujos lançamentos upstream fornecem executáveis independentes, como:

```text
myapp
myapp-linux-x86_64
myapp-v1.4.2-linux-amd64
```

Quando o LinuxToys instala um aplicativo de binário único, ele automaticamente:

* cria um diretório para o aplicativo em:

  ```text
  ~/.local/linuxtoys/apps/<nome do aplicativo>/
  ```

* copia o binário baixado para esse diretório;

* marca o binário como executável;

* cria um atalho no menu de aplicativos usando o nome, a descrição e o ícone da lista de repositório;

* registra a instalação no sistema de transações do LinuxToys para que ela possa ser revertida normalmente.

Em listas de repositório, binários únicos podem ser instalados a partir de um lançamento do GitHub ou diretamente de uma URL.

### Instalando um Binário a Partir de um Lançamento do GitHub

Use:

```json
"type": "bin"
```

O campo `repo` deve apontar para o repositório do aplicativo no GitHub, enquanto `package-name` deve conter o **nome exato do arquivo do lançamento** que contém o executável.

Por exemplo:

```json
{
  "name": "Example App",
  "description": "A standalone example application.",
  "category": "utilities",
  "repo": "https://github.com/example/example",
  "type": "bin",
  "package-name": "example-linux-x86_64",
  "icon": "example.svg"
}
```

O LinuxToys obterá o lançamento estável mais recente do GitHub, localizará o arquivo solicitado, fará seu download e o instalará como um aplicativo independente.

### O Nome do Arquivo Binário Deve Ser Informado Explicitamente

Ao contrário de formatos de pacote como `.deb`, `.rpm` ou `.AppImage`, binários independentes frequentemente **não possuem uma extensão que permita identificá-los**.

Por isso, o LinuxToys não pode determinar com segurança qual arquivo do lançamento corresponde ao binário do aplicativo de forma automática.

Desenvolvedores utilizando `"type": "bin"` devem, portanto, fornecer o nome exato do arquivo do lançamento através de `package-name`.

Por exemplo, se um lançamento contém:

```text
example-linux-x86_64
example-linux-aarch64
example.sha256
source.tar.gz
```

a entrada da lista de repositório deve selecionar explicitamente:

```json
"package-name": "example-linux-x86_64"
```

Curingas não devem ser utilizados para arquivos binários de lançamentos.

### Versões de Lançamento no Nome do Binário

Alguns projetos incluem a versão do lançamento diretamente no nome do arquivo binário.

Por exemplo, um lançamento upstream com a tag:

```text
v2.4.1
```

pode conter:

```text
example-v2.4.1-linux-x86_64
```

Para esses casos, o LinuxToys disponibiliza:

```text
$APP_GIT_VERSION
```

dentro do nome do arquivo binário.

Portanto, é possível escrever:

```json
{
  "name": "Example App",
  "description": "A standalone example application.",
  "category": "utilities",
  "repo": "https://github.com/example/example",
  "type": "bin",
  "package-name": "example-$APP_GIT_VERSION-linux-x86_64"
}
```

O LinuxToys determinará primeiro a versão do lançamento estável mais recente e substituirá `$APP_GIT_VERSION` antes de localizar o arquivo.

Isso evita a necessidade de atualizar a entrada da lista de repositório sempre que o upstream publicar uma nova versão.

> `APP_GIT_VERSION` corresponde à tag do lançamento no GitHub. Portanto, se o upstream utilizar tags como `v2.4.1`, o `v` fará parte do valor.

### Instalando um Binário Diretamente de uma URL

Um binário independente também pode ser instalado através do tipo regular `"url"` das listas de repositório.

Use a chave `bin` dentro de `urls`:

```json
{
  "name": "Example App",
  "description": "A standalone example application.",
  "category": "utilities",
  "repo": "https://example.org",
  "type": "url",
  "urls": {
    "bin": "https://example.org/releases/example-linux-x86_64"
  },
  "icon": "example.svg"
}
```

O LinuxToys fará o download do arquivo através do seu mecanismo normal de downloads por URL e, em seguida, o instalará utilizando o mesmo procedimento para aplicativos de binário único.

Isso é especialmente útil para projetos que publicam executáveis independentes fora dos Lançamentos do GitHub.

### Entradas Específicas por Arquitetura

Caso o upstream publique binários separados para diferentes arquiteturas de CPU, a entrada do repositório deve selecionar o arquivo correto para os sistemas suportados pela entrada.

Por exemplo:

```text
example-linux-x86_64
example-linux-aarch64
```

Uma entrada de repositório destinada somente a sistemas x86-64 deve referenciar:

```json
"package-name": "example-linux-x86_64"
```

e utilizar as restrições apropriadas de hardware ou compatibilidade das listas de repositório quando necessário.

O LinuxToys pode utilizar informações de arquitetura presentes nos nomes dos arquivos do lançamento ao localizar os arquivos, mas o desenvolvedor ainda deve identificar explicitamente o binário desejado.

### Integração com o Menu de Aplicativos

Instalações de binário único recebem automaticamente um atalho no menu de aplicativos.

O atalho utiliza os mesmos metadados já fornecidos pela entrada do repositório:

* `name` se torna o nome de exibição do aplicativo;
* a `description` traduzida, quando disponível, se torna a descrição do aplicativo;
* `icon` se torna o ícone do aplicativo;
* o executável instalado se torna o comando utilizado pelo atalho.

Consequentemente, desenvolvedores de listas de repositório normalmente **não precisam** fornecer um script de pós-instalação apenas para criar um arquivo `.desktop` para um aplicativo independente.

O binário é instalado em:

```text
~/.local/linuxtoys/apps/<nome do aplicativo>/
```

e o atalho gerado aponta para a cópia instalada, e não para o arquivo temporário utilizado durante o download.

### Escolhendo Entre `bin` e `url`

Use `"type": "bin"` quando:

* o aplicativo estiver hospedado nos Lançamentos do GitHub;
* o upstream distribuir o aplicativo como um único arquivo executável;
* for possível identificar o arquivo do lançamento pelo seu nome exato.

Use `"type": "url"` com:

```json
"urls": {
  "bin": "..."
}
```

quando:

* o executável independente estiver disponível através de uma URL direta e estável;
* o projeto não utilizar os Lançamentos do GitHub para distribuição;
* ou você quiser explicitamente que o LinuxToys faça o download a partir de outra fonte.

---

<a id="tarball-package"></a>

## Aplicativos em Tarball

O tipo `tar` é destinado a aplicativos distribuídos como **tarballs binários pré-compilados** por meio de releases do GitHub ou Codeberg. Ele permite que o LinuxToys instale softwares que não fornecem um pacote nativo, Flatpak ou AppImage, mas distribuem o aplicativo pronto para execução em um arquivo `.tar.gz` ou `.tar.xz`.

> **Observação:** `tar` é destinado a releases binários de aplicativos, não a arquivos de código-fonte, e requer, **obrigatoriamente**, um script pós-instalação para finalizar a configuração. Instalações deste tipo exportam a variável **`LINUXTOYS_TARBALL_DIR`** que aponta para o nome de diretório final da tarball extraída para uso no script pós-instalação. Se estiver distribuindo um aplicativo neste formato, pode ter interesse também em [criar um atalho `.desktop` para o menu de aplicativos automaticamente](corelibraries.pt-BR.html#app-shortcuts).

### Releases do GitHub e Codeberg

Para um aplicativo distribuído como um tarball anexado a um release do GitHub ou Codeberg, use:

```json
{
  "name": "myapp",
  "type": "tar",
  "repo": "https://github.com/example/myapp"
}
```

Internamente, isso faz com que o LinuxToys utilize o instalador de releases no modo tarball:

```bash
pkg_fromrelease --tar "https://github.com/example/myapp"
```

O release mais recente é consultado e o LinuxToys procura especificamente por um arquivo `.tar.gz` ou `.tar.xz` compatível entre seus assets.

Os arquivos de código-fonte gerados automaticamente pelo GitHub não são considerados, pois não fazem parte da lista de assets enviados ao release. Assets de release identificados como arquivos de código-fonte também são filtrados. Portanto, os desenvolvedores devem fornecer o **tarball do aplicativo compilado como um asset propriamente dito do release**.

As informações de arquitetura presentes nos nomes dos assets são respeitadas. Por exemplo:

```text
myapp-2.4.0-x86_64.tar.xz
myapp-2.4.0-aarch64.tar.xz
myapp-2.4.0-source.tar.gz
```

Em um sistema x86-64, o LinuxToys selecionará o arquivo `x86_64` do aplicativo, enquanto excluirá o arquivo destinado à arquitetura incompatível e o arquivo de código-fonte.

### URLs Diretas

Tarballs hospedados diretamente pelo desenvolvedor ou pela infraestrutura do projeto podem utilizar o tipo `url`:

```json
{
  "name": "myapp",
  "type": "url",
  "urls": {
    "tar": "https://example.com/releases/myapp.tar.xz"
  }
}
```

Isso chama o instalador por URL no modo tarball:

```bash
pkg_fromurl --tar "https://example.com/releases/myapp.tar.xz"
```

A URL pode apontar diretamente para o arquivo ou utilizar um redirecionamento HTTP. O LinuxToys resolve o nome do arquivo baixado antes de determinar o formato do pacote.

O arquivo resolvido deve estar em um formato de tarball compatível.

### Formatos Compatíveis

Atualmente, o manipulador de tarballs aceita:

```text
.tar.gz
.tar.xz
```

Outros formatos de arquivo não devem ser declarados utilizando `tar`.

### Estrutura do Arquivo

Os desenvolvedores podem empacotar o aplicativo dentro de um único diretório de nível superior ou colocar os arquivos do aplicativo diretamente na raiz do tarball.

Um tarball que já contém seu próprio diretório:

```text
MyApp/
├── bin/
│   └── myapp
├── lib/
└── resources/
```

é instalado diretamente como:

```text
~/.local/linuxtoys/apps/MyApp/
```

O LinuxToys detecta o diretório comum de nível superior existente e **não** cria um diretório adicional desnecessário, como `MyApp/MyApp/`.

Um tarball que contém arquivos soltos no nível raiz também é compatível:

```text
myapp
lib/
resources/
README.md
```

Nesse caso, o LinuxToys cria um diretório para o aplicativo utilizando o nome do arquivo, excluindo a extensão `.tar.gz` ou `.tar.xz`.

Por exemplo:

```text
myapp-2.4.0.tar.xz
```

resultaria em:

```text
~/.local/linuxtoys/apps/myapp-2.4.0/
```

Por esse motivo, recomenda-se que os desenvolvedores distribuam o conteúdo dentro de um diretório de nível superior com um nome adequado quando for necessário manter um nome de diretório de instalação estável entre diferentes releases.

### Atualizações

Executar novamente a mesma instalação por tarball é tratado como uma atualização.

O LinuxToys substitui o diretório de destino existente pelo aplicativo recém-extraído, em vez de mesclar o novo arquivo com a instalação anterior. Isso garante que arquivos removidos pelo upstream em versões mais recentes não permaneçam no sistema após uma atualização.

Portanto, desenvolvedores que distribuem releases sucessivos devem, preferencialmente, manter consistente o diretório de nível superior do tarball entre as versões:

```text
myapp/
```

em vez de:

```text
myapp-2.4.0/
myapp-2.5.0/
```

Um nome de diretório estável permite que releases posteriores substituam corretamente a instalação anterior.

### Tipos Específicos por Sistema Operacional

O tipo `tar` também pode ser selecionado através do mapeamento normal de `type` específico por sistema operacional. Por exemplo, um projeto pode utilizar um pacote nativo no Arch Linux enquanto distribui um tarball binário para os demais sistemas compatíveis:

```json
{
  "name": "myapp",
  "type": {
    "arch": "native",
    "all": "tar"
  },
  "package-name": {
    "arch": "myapp"
  },
  "repo": "https://github.com/example/myapp"
}
```

No Arch Linux e derivados, o LinuxToys utilizará o pacote nativo. Nos demais sistemas compatíveis, obterá o tarball binário a partir dos releases do projeto.

### Escolhendo entre `tar` e `url`

Use:

```json
"type": "tar"
```

quando o tarball binário for publicado como um asset dos releases do projeto no GitHub ou Codeberg e o LinuxToys deva acompanhar automaticamente novos releases.

Use:

```json
"type": "url"
```

com:

```json
"urls": {
  "tar": "https://example.com/application.tar.gz"
}
```

quando o arquivo estiver hospedado em uma URL fornecida diretamente pelo desenvolvedor.

Em ambos os casos, o arquivo deve conter um aplicativo já compilado e pronto para uso. A compilação de tarballs contendo código-fonte está fora do escopo do tipo `tar`.

---

## Utilizando URLs de Download Dinâmicas

As páginas de aplicativos não alteram a forma como um aplicativo é instalado. Portanto, elas podem ser combinadas com outras funcionalidades das listas de repositórios, incluindo URLs de download descobertas dinamicamente.

Por exemplo:

```json
{
  "name": "Example App",
  "repo": "https://example.org",
  "category": "office",
  "icon": "./example.svg",
  "type": "url",

  "urls": {
    "appimage": {
      "env": "URL"
    }
  },

  "overrides": {
    "pre": {
      "script": "./example-pre.sh"
    }
  },

  "descriptions": "descriptions.json",
  "screenshots": "screenshots/"
}
```

O script de pré-instalação pode descobrir a URL atual de download e exportá-la:

```bash
#!/usr/bin/env bash

# Determinar a URL apropriada da versão...
export URL="https://example.org/releases/latest/example.AppImage"
```

O LinuxToys então expande essa variável de ambiente quando `pkg_fromurl` é chamado.

Declarações de URLs dinâmicas exigem um hook de pré-instalação. Os nomes das variáveis de ambiente são declarados explicitamente utilizando a forma `{"env": "VARIABLE"}`, em vez de inserir variáveis do shell diretamente nas strings de URLs.

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

<a id="app-pages"></a>

## Páginas de Aplicativos

Entradas de listas de repositórios podem, opcionalmente, fornecer uma **página de aplicativo**. Isso permite apresentar mais informações sobre um aplicativo antes da instalação, incluindo uma descrição mais longa, capturas de tela e links opcionais para compra ou doação.

As páginas de aplicativos são destinadas a aplicativos que se beneficiam de uma apresentação mais completa do que a caixa de diálogo padrão de confirmação de instalação.

Se nenhum dos campos relacionados à página de aplicativo for fornecido, o LinuxToys ignora completamente essa página e segue o fluxo normal de instalação.

#### Exemplo Básico

Uma entrada com uma página de aplicativo pode ser semelhante a esta:

```json
{
  "name": "Example App",
  "repo": "https://example.org",
  "category": "office",
  "icon": "./example.svg",
  "type": "url",

  "urls": {
    "appimage": "https://example.org/releases/example.AppImage"
  },

  "description": "Uma descrição curta do aplicativo.",

  "long-description": "Uma descrição mais longa contendo informações adicionais sobre o aplicativo e seus recursos.",

  "screenshots": "screenshots/",

  "donate": "https://example.org/donate"
}
```

Quando o usuário seleciona essa entrada, o LinuxToys abre sua página de aplicativo em vez de exibir imediatamente a confirmação de instalação.

A página mantém o cabeçalho padrão do LinuxToys contendo o nome do aplicativo, sua descrição curta, informações do repositório e ícone. A área principal exibe a descrição mais longa e o visualizador de capturas de tela.

Selecionar **Instalar** continua pelo fluxo normal de instalação do LinuxToys.

### Descrições Longas

A `description` curta continua sendo o texto exibido pela interface normal do LinuxToys. A página do aplicativo pode fornecer adicionalmente uma descrição mais longa:

```json
"long-description": "Uma descrição detalhada do aplicativo, sua finalidade e seus principais recursos."
```

A forma com sublinhado também é aceita:

```json
"long_description": "Uma descrição detalhada."
```

Descrições longas também podem utilizar o sistema normal de traduções do LinuxToys:

```json
"description": "Descrição curta de fallback.",
"description_tag": "example_desc",

"long-description": "Descrição longa de fallback.",
"long-description_tag": "example_long_desc"
```

No entanto, para entradas de repositório com descrições mais extensas, recomenda-se utilizar um catálogo de descrições local do repositório.

### Traduções de Descrições Locais do Repositório

Listas de repositórios podem manter suas descrições de aplicativos separadas dos arquivos principais de tradução do LinuxToys colocando um catálogo JSON de descrições junto ao arquivo da lista de repositório.

Faça referência a ele com:

```json
"descriptions": "descriptions.json"
```

`description-file` também é aceito como um alias.

O arquivo de descrições deve estar localizado no **mesmo diretório que o JSON da lista de repositório**.

Por exemplo:

```text
scripts/lists/example/
├── repository.json
├── descriptions.json
├── example.svg
└── screenshots/
    ├── main.webp
    ├── editor.webp
    └── settings.webp
```

Um arquivo `descriptions.json` utiliza esta estrutura:

```json
{
  "description_tag": "example_desc",
  "description_long_tag": "example_long",

  "en": {
    "example_desc": "A short description of the application.",
    "example_long": "A longer description explaining the application and its main features."
  },

  "pt": {
    "example_desc": "Uma descrição curta do aplicativo.",
    "example_long": "Uma descrição mais longa explicando o aplicativo e seus principais recursos."
  }
}
```

`description_tag` identifica a descrição curta, enquanto `description_long_tag` identifica a descrição longa utilizada pela página do aplicativo.

O LinuxToys procura primeiro pelo idioma atualmente selecionado e utiliza o inglês como fallback quando uma tradução apropriada não está disponível.

Descrições inline e tags de tradução existentes continuam sendo suportadas, o que é útil durante a migração de um script existente do LinuxToys para uma entrada de lista de repositório.

### Capturas de Tela

Use `screenshots` para fornecer imagens para a página do aplicativo.

Há duas maneiras de fazer isso.

#### Diretório de Capturas de Tela

O método mais simples é apontar para um diretório:

```json
"screenshots": "screenshots/"
```

O LinuxToys carrega automaticamente os arquivos de imagem suportados diretamente dentro desse diretório.

Os formatos suportados são:

* `.png`
* `.jpg`
* `.jpeg`
* `.webp`
* `.svg`

Os arquivos são ordenados pelo nome, portanto os nomes dos arquivos também podem ser utilizados para controlar sua ordem:

```text
screenshots/
├── 01-main.webp
├── 02-editor.webp
└── 03-settings.webp
```

#### Capturas de Tela Individuais

Também é possível listar arquivos específicos:

```json
"screenshots": [
  "screenshots/main.webp",
  "screenshots/editor.webp",
  "screenshots/settings.webp"
]
```

Os caminhos das capturas de tela são relativos ao JSON da lista de repositório.

Por segurança, os caminhos das capturas de tela devem permanecer dentro da hierarquia `scripts/lists`. Caminhos que resolvam para locais externos a ela são rejeitados.

#### Visualizador de Capturas de Tela

Quando várias capturas de tela estão disponíveis, o LinuxToys as apresenta em um visualizador circular.

Os usuários podem navegar tanto para frente quanto para trás entre as imagens. Ao chegar a qualquer uma das extremidades, a navegação retorna para a outra extremidade:

```text
1 → 2 → 3 → 1
```

e:

```text
1 ← 2 ← 3 ← 1
```

As teclas de seta Esquerda e Direita do teclado também podem ser utilizadas enquanto a página do aplicativo estiver aberta.

### Links para Doação

Um link para doação pode ser adicionado com:

```json
"donate": "https://example.org/donate"
```

A forma de objeto também é aceita:

```json
"donate": {
  "url": "https://example.org/donate"
}
```

O LinuxToys exibe um botão **Doar** na página do aplicativo, que abre a URL especificada.

Somente URLs HTTP ou HTTPS válidas são aceitas.

### Aplicativos Pagos

Aplicativos que precisam ser adquiridos em vez de simplesmente baixados podem fornecer um link de compra e um preço:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99
}
```

O campo `price` define o preço base do aplicativo e é sempre especificado como um valor numérico em **dólares americanos (USD)**.

O LinuxToys exibe o preço diretamente no botão de compra, por exemplo:

```text
Comprar · $19.99
```

O botão de compra recebe destaque visual na página do aplicativo.

#### Preços Localizados

Além do preço base em dólares americanos, o desenvolvedor pode fornecer preços específicos para outras moedas por meio do campo `prices`:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99,
  "prices": {
    "BRL": 59.90,
    "EUR": 17.99,
    "GBP": 15.99
  }
}
```

As chaves de `prices` correspondem aos códigos internacionais de moeda fornecidos pelo `locale int_curr_symbol` do sistema, como `BRL`, `EUR` e `GBP`.

Quando o LinuxToys encontra uma moeda correspondente à configuração regional do sistema, utiliza o preço localizado em vez do preço base. O símbolo monetário exibido no botão é obtido automaticamente por meio do `locale currency_symbol`.

Por exemplo, em um sistema cuja configuração regional informe `BRL`, a configuração acima pode ser exibida como:

```text
Comprar · R$59.90
```

Já em um sistema configurado para `EUR`:

```text
Comprar · €17.99
```

Não é necessário adicionar uma entrada `USD` a `prices`. O campo `price` já representa o preço em dólares americanos e funciona como **fallback obrigatório**.

Se a moeda do sistema não estiver presente em `prices`, se as informações monetárias da configuração regional não puderem ser determinadas ou se `locale` não estiver disponível, o LinuxToys utiliza automaticamente o preço base em USD e o símbolo `$`.

Dessa forma, uma entrada pode oferecer preços localizados apenas para os mercados em que isso for desejado:

```json
"purchase": {
  "url": "https://example.org/buy",
  "price": 19.99,
  "prices": {
    "BRL": 59.90
  }
}
```

Nesse exemplo, usuários com `BRL` recebem o preço localizado de `R$59.90`, enquanto todos os demais recebem o preço base de `$19.99`.

Um URL de compra também pode ser fornecido sem um preço:

```json
"purchase": {
  "url": "https://example.org/buy"
}
```

Nesse caso, o LinuxToys simplesmente exibe **Comprar**.

Quando links de compra e doação são fornecidos simultaneamente, ambos os botões são exibidos, com a ação de compra recebendo o destaque principal.

### Exemplo Completo

Uma entrada de repositório mais completa pode, portanto, ser semelhante a esta:

```json
[
  {
    "name": "Example App",
    "repo": "https://example.org",
    "category": "office",
    "icon": "./example.svg",
    "type": "url",
    "urls": {
      "appimage": "https://example.org/releases/example.AppImage"
    },
    "descriptions": "descriptions.json",
    "screenshots": "screenshots/",
    "purchase": {
      "url": "https://example.org/purchase",
      "price": 14.99,
      "prices": {
        "BRL": 44.90,
        "EUR": 12.99
      }
    },
    "donate": "https://example.org/donate"
  }
]
```

Com a seguinte estrutura de diretórios:

```text
scripts/lists/example/
├── repository.json
├── descriptions.json
├── example.svg
└── screenshots/
    ├── 01-main.webp
    ├── 02-project.webp
    └── 03-settings.webp
```

E:

```json
{
  "description_tag": "example_desc",
  "description_long_tag": "example_long",

  "en": {
    "example_desc": "A short description of Example App.",
    "example_long": "A detailed explanation of Example App, its purpose, and the features available to the user."
  },

  "pt": {
    "example_desc": "Uma descrição curta do Example App.",
    "example_long": "Uma explicação detalhada do Example App, sua finalidade e os recursos disponíveis para o usuário."
  }
}
```

### Quando uma Página de Aplicativo é Exibida?

Uma página de aplicativo é habilitada automaticamente quando a entrada fornece pelo menos um recurso de página de aplicativo:

* uma descrição longa;
* uma ou mais capturas de tela válidas;
* uma URL de compra; ou
* uma URL de doação.

Não é necessário habilitá-la explicitamente com uma opção adicional.

Se nenhum desses elementos estiver presente, selecionar o aplicativo segue o fluxo padrão de instalação do LinuxToys.

### Comportamento em Checklists

As páginas de aplicativos afetam apenas a ativação individual de um aplicativo.

Se um aplicativo for selecionado individualmente em uma checklist, sua página de aplicativo será aberta normalmente quando estiver disponível.

Quando o usuário realiza uma **instalação de múltiplos aplicativos por checklist**, as páginas de aplicativos são ignoradas intencionalmente. O LinuxToys segue diretamente para o fluxo normal de instalação em lote, evitando que várias páginas de aplicativos interrompam uma operação de checklist.

Abrir a página de um aplicativo individual não limpa as seleções já feitas pelo usuário na checklist.

### Recomendações

Mantenha a descrição curta concisa, pois ela é utilizada na interface normal do LinuxToys e no cabeçalho da página do aplicativo. Utilize a descrição longa para fornecer contexto adicional, apresentar os principais recursos, informar sobre compatibilidade ou incluir outros detalhes que ajudem o usuário a decidir se deseja instalar o aplicativo.

Para aplicativos com descrições longas traduzidas, dê preferência a um arquivo `descriptions.json` local do repositório. Isso mantém textos específicos do aplicativo fora dos arquivos principais de tradução do LinuxToys e permite que a lista de repositório, as capturas de tela, o ícone e as descrições sejam mantidos em conjunto.

Uma página de aplicativo é opcional. Pacotes simples que precisam apenas de um nome e uma descrição curta geralmente devem continuar utilizando o fluxo padrão de instalação.

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
