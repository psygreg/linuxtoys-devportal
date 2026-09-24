# Listagens de Repositório

As listagens de repositório são o formato declarativo do LinuxToys para
softwares que não podem ser tratados inteiramente pelo catálogo
AppStream normal.

Se um aplicativo já estiver disponível pelo Flathub ou pelos
repositórios nativos de uma distribuição, normalmente ele deve ser
descoberto e instalado pelo AppStream. As listagens de repositório são
destinadas a softwares distribuídos por fontes como lançamentos Git,
AppImages, URLs de download direto, tarballs, binários independentes ou
repositórios de código-fonte que exigem uma etapa de compilação.

Overlays do AppStream são documentados separadamente. Eles estendem um
aplicativo AppStream existente e não devem ser tratados como listagens
de repositório normais.

## Início rápido

Uma listagem de repositório é um objeto JSON. No mínimo, uma entrada
normal precisa de um nome, URL do repositório/fonte, categoria e
descrição curta.

Para um aplicativo publicado por lançamentos do GitHub, uma entrada
mínima pode ser assim:

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Uma descrição curta do Meu App."
}
```

`git` é o tipo de instalação padrão, portanto `"type": "git"` pode ser
omitido.

O LinuxToys usa as informações de lançamento do repositório para
localizar um asset compatível com a arquitetura e o sistema atuais.
Quando a escolha automática não for específica o suficiente,
`package-name` pode selecionar o asset de lançamento desejado.

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Uma descrição curta do Meu App.",
  "package-name": "MyApp-*.AppImage"
}
```

Comece com a menor listagem que descreva corretamente o aplicativo.
Regras de compatibilidade, dependências, hooks, serviços e metadados
avançados da página do aplicativo podem ser adicionados apenas quando
forem necessários.

## Escolhendo uma fonte de instalação

A decisão mais importante em uma listagem de repositório é de onde o
LinuxToys deve obter o aplicativo.

### Lançamentos Git

Use o tipo `git` padrão para aplicativos distribuídos como assets de
lançamento de um repositório Git compatível.

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Um aplicativo portátil para desktop."
}
```

O campo `repo` identifica o projeto, e não um arquivo de lançamento
específico. O LinuxToys resolve o lançamento adequado mais recente e
escolhe um asset compatível.

As URLs de projeto compatíveis são repositórios HTTPS hospedados no
GitHub, Codeberg ou GitLab.

Quando vários assets de lançamento puderem corresponder, use
`package-name` como seletor de asset:

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Um aplicativo portátil para desktop.",
  "package-name": "MyApp-*.AppImage"
}
```

Para entradas `git`, `package-name` é opcional e pode ser um nome de
asset ou glob. Ele também pode variar de acordo com o sistema
operacional:

``` json
{
  "package-name": {
    "arch": "MyApp-*-arch.AppImage",
    "all": "MyApp-*.AppImage"
  }
}
```

O valor `all` é usado como fallback quando nenhuma chave de sistema
operacional mais específica corresponde.

### AppImages

Um AppImage publicado como lançamento Git normalmente não precisa de um
tipo de instalação especial. Use uma listagem `git` e, quando
necessário, selecione o asset AppImage com `package-name`.

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Um aplicativo AppImage portátil.",
  "package-name": "MyApp-*.AppImage"
}
```

Se o AppImage for distribuído por uma URL direta estável em vez de um
lançamento Git, use `url`:

``` json
{
  "name": "Meu App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "Um aplicativo AppImage portátil.",
  "type": "url",
  "urls": {
    "appimage": "https://example.com/download/MyApp.AppImage"
  }
}
```

O campo `repo` ainda identifica o projeto ou a fonte upstream do
aplicativo. O arquivo que será baixado pertence a `urls`.

### URLs diretas

Use `type: "url"` quando o LinuxToys deve baixar um pacote de uma URL
explícita em vez de descobri-lo por um lançamento Git.

``` json
{
  "name": "Meu App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "Um aplicativo distribuído por download direto.",
  "type": "url",
  "urls": {
    "appimage": "https://example.com/download/MyApp.AppImage"
  }
}
```

O objeto `urls` descreve os formatos de pacote disponíveis. O LinuxToys
entende estas chaves:

``` text
deb
rpm
pacman
pkg.tar.zst
flatpak
appimage
tar
bin
```

Quando mais de uma URL compatível é fornecida, o LinuxToys prefere um
pacote nativo apropriado para a distribuição atual e depois usa formatos
portáteis como fallback nesta ordem:

``` text
appimage → flatpak → tar → bin
```

Isso permite descrever vários downloads upstream em uma única entrada:

``` json
{
  "name": "Meu App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "Um aplicativo com vários pacotes upstream.",
  "type": "url",
  "urls": {
    "deb": "https://example.com/download/my-app.deb",
    "rpm": "https://example.com/download/my-app.rpm",
    "appimage": "https://example.com/download/MyApp.AppImage"
  }
}
```

Para aplicativos já distribuídos por repositórios nativos normais ou
pelo Flathub, prefira o AppStream em vez de recriar esse caminho de
distribuição como uma listagem de repositório.

#### URLs de download dinâmicas

Alguns projetos geram uma URL de download dinamicamente. Um valor de URL
pode referenciar uma variável de ambiente exportada por um hook de
pré-instalação:

``` json
{
  "type": "url",
  "urls": {
    "appimage": {
      "env": "DOWNLOAD_URL"
    }
  },
  "overrides": {
    "pre": "export DOWNLOAD_URL=\"https://example.com/generated/MyApp.AppImage\""
  }
}
```

A forma com variável de ambiente só é aceita quando existe um hook de
pré-instalação válido. Use-a apenas quando uma URL estável normal não
puder descrever o download.

### Tarballs

Use `type: "tar"` para um tarball publicado como lançamento Git:

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Um aplicativo distribuído como arquivo de lançamento.",
  "type": "tar",
  "package-name": "my-app-*.tar.gz",
  "overrides": {
    "post": {
      "script": "my-app/post-install.sh"
    }
  }
}
```

Para um tarball hospedado em uma URL direta, use `type: "url"`:

``` json
{
  "name": "Meu App",
  "repo": "https://example.com/my-app",
  "category": "utilities",
  "description": "Um aplicativo distribuído como arquivo para download.",
  "type": "url",
  "urls": {
    "tar": "https://example.com/download/my-app.tar.gz"
  },
  "overrides": {
    "post": {
      "script": "my-app/post-install.sh"
    }
  }
}
```

Uma instalação por tarball apenas extrai os arquivos do aplicativo. Por
isso, o LinuxToys exige um hook de pós-instalação válido para entradas
tarball. Use esse hook para a integração de que o aplicativo extraído
precisa, como criar um lançador, entrada de desktop ou link simbólico.

### Binários independentes

Use `type: "bin"` para um executável independente publicado como
lançamento Git.

``` json
{
  "name": "Minha Ferramenta",
  "repo": "https://github.com/example/my-tool",
  "category": "utilities",
  "description": "Um utilitário de linha de comando independente.",
  "type": "bin",
  "package-name": "my-tool"
}
```

Ao contrário do seletor usado por `git` e `tar`, uma entrada `bin` exige
o nome exato do arquivo do asset. Globs e caminhos não são aceitos.

Para um binário hospedado diretamente, use `url`:

``` json
{
  "name": "Minha Ferramenta",
  "repo": "https://example.com/my-tool",
  "category": "utilities",
  "description": "Um utilitário de linha de comando independente.",
  "type": "url",
  "urls": {
    "bin": "https://example.com/download/my-tool"
  }
}
```

### Compilando com Make

Use `type: "make"` quando o aplicativo precisar ser compilado do
código-fonte com `make`.

Por padrão, o LinuxToys clona o repositório Git, compila o projeto e usa
`sudo make install` como comando de instalação.

``` json
{
  "name": "Minha Ferramenta",
  "repo": "https://github.com/example/my-tool",
  "category": "utilities",
  "description": "Uma ferramenta compilada do código-fonte.",
  "type": "make"
}
```

Um alvo de instalação personalizado pode ser declarado com
`make-command`:

``` json
{
  "type": "make",
  "make-command": "make install-user"
}
```

O comando deve conter um alvo de instalação que o LinuxToys possa mapear
para o alvo de desinstalação correspondente durante a reversão.

Uma compilação Make também pode usar um tarball de lançamento em vez de
um clone Git:

``` json
{
  "name": "Minha Ferramenta",
  "repo": "https://github.com/example/my-tool",
  "category": "utilities",
  "description": "Uma ferramenta compilada de um tarball de lançamento.",
  "type": "make",
  "make-source": "tar",
  "package-name": "my-tool-*.tar.gz"
}
```

`make-source` aceita `git` ou `tar`.

## Campos obrigatórios

Entradas normais de repositório exigem estes campos:

### `name`

O nome de exibição mostrado pelo LinuxToys.

``` json
"name": "Meu App"
```

Os nomes devem ser únicos entre as listagens de repositório. O LinuxToys
também deriva um ID interno estável da entrada a partir do nome de
exibição.

### `repo`

A URL do projeto ou fonte upstream do aplicativo.

``` json
"repo": "https://github.com/example/my-app"
```

Para tipos de instalação baseados em Git, esse também é o repositório
que o LinuxToys usa para descobrir lançamentos ou obter o código-fonte.

Para entradas `url`, as URLs dos pacotes para download pertencem a
`urls`; `repo` continua representando a identidade upstream do
aplicativo.

### `category`

A categoria do LinuxToys em que o aplicativo deve aparecer.

``` json
"category": "utilities"
```

Use um identificador de categoria existente no LinuxToys.

### `description`

Uma descrição curta exibida nas listas de aplicativos e nos resultados
de busca.

``` json
"description": "Um utilitário de desktop rápido e leve."
```

Um catálogo de descrições localizadas pode ser usado em vez de manter
todas as traduções diretamente na listagem. Consulte
[Localização](#localização).

## Compatibilidade

As listagens de repositório são filtradas antes de serem exibidas ao
usuário. Adicione restrições de compatibilidade apenas quando o
aplicativo ou método de instalação realmente precisar delas.

### Sistemas operacionais

Use `os` para restringir uma entrada a famílias de compatibilidade
específicas do LinuxToys:

``` json
"os": ["debian", "ubuntu", "fedora"]
```

As chaves compatíveis são:

``` text
debian
ubuntu
cachy
arch
steamos
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

Valores positivos formam uma lista de permissões.

Exclusões começam com `!`:

``` json
"os": ["!steamos"]
```

Uma lista contendo apenas exclusões significa "todos os sistemas
compatíveis, exceto estes". Se valores positivos e negativos forem
misturados, as exclusões sempre têm precedência.

Não inclua e exclua a mesma chave em uma única declaração.

### Tipos de instalação por sistema operacional

O campo `type` pode ser um mapeamento quando um aplicativo realmente
precisa de métodos de instalação diferentes em sistemas diferentes:

``` json
"type": {
  "arch": "make",
  "all": "git"
}
```

`all` é o fallback. Prefira um único tipo de instalação sempre que
possível.

### Nomes de pacote ou asset por sistema operacional

Campos que resolvem nomes de pacotes ou seletores de lançamento também
podem usar mapeamentos por sistema operacional quando compatível:

``` json
"package-name": {
  "arch": "MyApp-*-arch.AppImage",
  "all": "MyApp-*.AppImage"
}
```

### Ambientes de desktop

Use `desktop` quando um aplicativo ou integração fizer sentido apenas
para determinadas famílias de desktop:

``` json
"desktop": ["gnome", "plasma"]
```

Os valores compatíveis são:

``` text
gnome
plasma
hyprland
sway
other
```

### systemd

Use `systemd` apenas quando a instalação exigir ou excluir
explicitamente o systemd:

``` json
"systemd": "yes"
```

ou:

``` json
"systemd": "no"
```

Omitir o campo é neutro.

Declarar serviços também exige implicitamente um ambiente compatível com
systemd.

### WSL

Use `wsl` para tornar uma entrada exclusiva para WSL ou exclusiva para
ambientes que não sejam WSL:

``` json
"wsl": "yes"
```

ou:

``` json
"wsl": "no"
```

### Contêineres

Entradas de repositório são permitidas em contêineres compatíveis por
padrão.

Para impedir explicitamente que uma entrada apareça em contêineres:

``` json
"container": "deny"
```

Os valores aceitos são `allow` e `deny`.

Fluxos de instalação Flatpak e AppImage não são compatíveis dentro de
contêineres independentemente dessa configuração.

### Hardware

Requisitos de hardware podem ser expressos com valores de
compatibilidade `gpu` e `cpu`:

``` json
"hardware": {
  "gpu": ["amd"],
  "cpu": ["x86_64"]
}
```

Use restrições de hardware apenas quando o aplicativo ou método de
instalação realmente depender delas.

## Dependências

As dependências são instaladas antes do aplicativo principal.

Uma dependência pode ser `native` ou `flathub`.

### Dependências nativas

``` json
"dependencies": [
  {
    "type": "native",
    "package-name": "ffmpeg"
  }
]
```

Os nomes dos pacotes nativos podem variar de acordo com o sistema
operacional:

``` json
"dependencies": [
  {
    "type": "native",
    "package-name": {
      "debian": "libexample-dev",
      "fedora": "example-devel",
      "arch": "example"
    }
  }
]
```

Um nome de pacote também pode ser uma lista quando vários pacotes forem
necessários:

``` json
"dependencies": [
  {
    "type": "native",
    "package-name": ["git", "curl", "make"]
  }
]
```

### Dependências do Flathub

``` json
"dependencies": [
  {
    "type": "flathub",
    "package-name": "org.example.Runtime"
  }
]
```

Dependências do Flathub exigem um ambiente systemd compatível.

As dependências são pré-requisitos do aplicativo definido pela listagem
de repositório. Elas não transformam aplicativos nativos ou do Flathub
comuns em listagens de repositório.

## Hooks e overrides de instalação

O objeto opcional `overrides` personaliza o fluxo de instalação gerado.

As chaves compatíveis para listagens de repositório normais são:

``` text
pre
post
flatpak
skip-user
```

### Hooks de pré-instalação

Um hook de pré-instalação é executado antes dos comandos de instalação
gerados.

Para uma operação pequena, shell inline pode ser usado:

``` json
"overrides": {
  "pre": "export EXAMPLE_MODE=1"
}
```

Para uma lógica mais substancial, referencie um script armazenado junto
à listagem de repositório:

``` json
"overrides": {
  "pre": {
    "script": "my-app/pre-install.sh"
  }
}
```

Os caminhos dos scripts são resolvidos relativamente ao arquivo JSON e
devem permanecer dentro da árvore de listagens de repositório.

### Hooks de pós-instalação

Hooks de pós-instalação usam a mesma sintaxe:

``` json
"overrides": {
  "post": {
    "script": "my-app/post-install.sh"
  }
}
```

Eles são executados depois dos comandos de instalação gerados.

Instalações por tarball exigem um hook de pós-instalação porque extrair
um arquivo, por si só, não define como o aplicativo deve ser integrado
ao sistema do usuário.

### Overrides de permissões Flatpak

Uma listagem de repositório pode declarar overrides de permissões
Flatpak quando seu fluxo de instalação inclui conteúdo Flatpak.

``` json
"overrides": {
  "flatpak": [
    {
      "scope": "user",
      "type": "filesystem",
      "setting": "xdg-download",
      "target": "org.example.App"
    }
  ]
}
```

Os escopos compatíveis são `user` e `system`.

Os tipos de override compatíveis incluem:

``` text
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

### Forçando o escopo de sistema do Flatpak

Quando as operações Flatpak de uma listagem não puderem usar o escopo de
usuário:

``` json
"overrides": {
  "skip-user": true
}
```

Use isso apenas quando o escopo de sistema for realmente necessário.

## Serviços

Listagens de repositório podem pedir ao LinuxToys para habilitar e
iniciar serviços systemd depois da instalação.

Uma string ou lista usa o escopo de sistema por padrão:

``` json
"services": "my-app.service"
```

``` json
"services": [
  "my-app.service",
  "my-app-helper.service"
]
```

Para serviços de sistema e usuário explícitos:

``` json
"services": {
  "system": ["my-app.service"],
  "user": ["my-app-session.service"]
}
```

Se o sufixo da unidade for omitido, `.service` é adicionado
automaticamente.

Serviços de sistema exigem elevação de privilégios. Serviços de usuário
são habilitados pela instância systemd do usuário.

## Metadados da página do aplicativo

Aplicativos definidos por listagens de repositório podem ter o mesmo
tipo de apresentação rica que os usuários esperam do restante do
LinuxToys.

Esses campos afetam a apresentação e não substituem a fonte de
instalação.

### Desenvolvedor

``` json
"developer": "Example Software"
```

Quando possível, o LinuxToys pode derivar o namespace do
desenvolvedor/projeto a partir da URL do repositório se esse campo for
omitido.

### Licença

``` json
"license": "GPL-3.0"
```

Mantenha o identificador de licença exibido conciso.

### Descrições longas

Uma descrição longa pode ser escrita diretamente:

``` json
"long-description": "Meu App oferece um fluxo de trabalho completo para..."
```

Para conteúdo mais extenso, aponte para um arquivo Markdown armazenado
junto à listagem de repositório:

``` json
"long-description": "my-app/description.md"
```

Os caminhos Markdown são resolvidos relativamente ao arquivo JSON e
podem usar subdiretórios, mas não podem sair da árvore de diretórios da
listagem.

### Capturas de tela

`screenshots` pode apontar para uma imagem:

``` json
"screenshots": "my-app/screenshots/main.webp"
```

ou várias:

``` json
"screenshots": [
  "my-app/screenshots/main.webp",
  "my-app/screenshots/settings.webp"
]
```

Também pode apontar para um diretório. O LinuxToys carregará os arquivos
de imagem compatíveis desse diretório em ordem alfabética.

Os formatos de captura de tela compatíveis são PNG, JPEG, WebP e SVG.

### Ícones

Um valor simples de ícone pode se referir a um nome de ícone GTK ou ao
resolvedor de ícones existente do LinuxToys:

``` json
"icon": "applications-utilities"
```

Uma listagem de repositório também pode usar um SVG ou PNG armazenado
abaixo de `scripts/lists/`:

``` json
"icon": "my-app/icon.svg"
```

Caminhos relativos de ícones são resolvidos a partir do arquivo JSON que
contém a entrada e não podem sair da árvore de listagens de repositório.

### Doações

``` json
"donate": "https://example.com/donate"
```

Uma URL de doação também pode ser representada como um objeto contendo
`url`.

### Compras e assinaturas

Metadados comerciais podem adicionar ações de compra e assinatura à
página de um aplicativo de repositório.

Uma compra única:

``` json
"purchase": {
  "url": "https://example.com/buy",
  "price": 49.99
}
```

Uma assinatura:

``` json
"purchase": {
  "url": "https://example.com/subscribe",
  "sub_price": 9.99
}
```

As duas opções podem ser oferecidas juntas:

``` json
"purchase": {
  "url": "https://example.com/pricing",
  "price": 49.99,
  "sub_price": 9.99
}
```

O LinuxToys também oferece suporte a preços localizados, níveis de
compra, níveis de assinatura e períodos de assinatura. Mantenha modelos
comerciais simples quando possível; use as formas com níveis apenas
quando os preços reais do aplicativo exigirem isso.

Os metadados comerciais descrevem o destino de compra ou assinatura do
próprio desenvolvedor. O LinuxToys não se torna o processador do
pagamento.

## Localização

Listagens de repositório podem manter suas descrições voltadas ao
usuário separadas dos metadados de instalação.

Uma listagem pode referenciar um catálogo JSON de descrições no mesmo
diretório:

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "descriptions": "descriptions.json",
  "description_tag": "my_app_desc",
  "long-description_tag": "my_app_long"
}
```

O catálogo de descrições deve ser um arquivo JSON no mesmo diretório do
JSON da listagem de repositório.

Um catálogo pode fornecer traduções como:

``` json
{
  "description_tag": "my_app_desc",
  "description_long_tag": "my_app_long",
  "en": {
    "my_app_desc": "A short description.",
    "my_app_long": "my-app/description.md"
  },
  "pt-BR": {
    "my_app_desc": "Uma descrição curta.",
    "my_app_long": "my-app/description-br.md"
  }
}
```

Para entradas localizadas do catálogo, o LinuxToys procura primeiro pelo
locale atual, depois pelo idioma-base e, por fim, pelo inglês.

Um valor localizado de descrição longa pode apontar para um arquivo
Markdown, permitindo que cada idioma use um documento separado.

## Um exemplo completo

O exemplo a seguir combina um AppImage hospedado no Git com metadados de
compatibilidade, uma dependência nativa, conteúdo rico para a página do
aplicativo e um link de doação:

``` json
{
  "name": "Meu App",
  "repo": "https://github.com/example/my-app",
  "category": "utilities",
  "description": "Um aplicativo portátil para desktop.",
  "package-name": "MyApp-*.AppImage",
  "icon": "my-app/icon.svg",
  "license": "GPL-3.0",
  "os": ["!steamos"],
  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": "example-helper",
        "ubuntu": "example-helper",
        "fedora": "example-helper",
        "arch": "example-helper",
        "all": "example-helper"
      }
    }
  ],
  "long-description": "my-app/description.md",
  "screenshots": "my-app/screenshots",
  "donate": "https://example.com/donate"
}
```

Não use um exemplo completo como se fosse um modelo que precisa ser
totalmente preenchido. A maioria dos aplicativos deve precisar de bem
menos campos.

## Estrutura do repositório

Os recursos de uma listagem de repositório podem ficar junto ao JSON que
os declara.

Por exemplo:

``` text
scripts/
└── lists/
    └── my-app/
        ├── app.json
        ├── descriptions.json
        ├── icon.svg
        ├── description.md
        ├── description-br.md
        ├── pre-install.sh
        ├── post-install.sh
        └── screenshots/
            ├── main.webp
            └── settings.webp
```

Ícones locais, capturas de tela, descrições Markdown e scripts de hook
são resolvidos com verificações de segurança de caminho para impedir que
as entradas saiam da árvore de listagens de repositório.

## Diretrizes de projeto

Prefira metadados declarativos a lógica shell. Se o LinuxToys já tiver
um campo para o que o aplicativo precisa, use esse campo em vez de
reproduzir o comportamento em um hook.

Prefira AppStream para aplicativos já representados pelo Flathub ou por
repositórios nativos compatíveis.

Prefira a descoberta de lançamentos Git quando o upstream publicar
assets de lançamento adequados. Use entradas `url` diretas quando o
local de download precisar ser declarado explicitamente.

Use restrições de compatibilidade apenas quando necessário. Uma
declaração `os`, desktop, hardware, WSL, contêiner ou systemd
desnecessariamente restrita impede que usuários com sistemas compatíveis
vejam o aplicativo.

Use hooks pre/post para as partes da instalação que não podem ser
expressas declarativamente. Mantenha esses hooks pequenos e fáceis de
auditar.

Trate a página do aplicativo como parte da integração. Uma descrição
útil, capturas de tela, informações de licença e links do desenvolvedor
ajudam os usuários a entender o aplicativo antes de instalá-lo.

## Referência de campos

| Campo | Finalidade |
| --- | --- |
| `name` | Nome de exibição do aplicativo. |
| `repo` | URL do projeto/fonte upstream. |
| `category` | Identificador de categoria do LinuxToys. |
| `description` | Descrição curta do aplicativo. |
| `type` | Tipo de instalação; o padrão é `git`. |
| `package-name` | Nome do pacote, asset binário exato ou seletor de asset de lançamento, dependendo de `type`. |
| `urls` | URLs diretas de pacotes/downloads para entradas `url`. |
| `make-source` | Fonte `git` ou `tar` para uma entrada `make`. |
| `make-command` | Comando de instalação Make personalizado. |
| `os` | Regras de inclusão/exclusão de compatibilidade por sistema operacional. |
| `desktop` | Compatibilidade com ambientes de desktop. |
| `systemd` | Exige ou exclui systemd. |
| `wsl` | Restrição exclusiva para WSL ou para ambientes que não sejam WSL. |
| `container` | Permite ou impede execução em contêineres. |
| `hardware` | Requisitos de compatibilidade de CPU/GPU. |
| `dependencies` | Pré-requisitos nativos ou do Flathub. |
| `overrides` | Hooks pre/post, overrides Flatpak e comportamento do escopo Flatpak. |
| `services` | Unidades systemd de sistema/usuário a habilitar e iniciar. |
| `icon` | Ícone GTK/LinuxToys ou SVG/PNG local do repositório. |
| `license` | Identificador curto de licença. |
| `developer` | Desenvolvedor ou empresa exibido na página do aplicativo. |
| `long-description` | Texto de descrição rica ou arquivo Markdown local do repositório. |
| `screenshots` | Capturas de tela locais do repositório ou diretório de capturas. |
| `donate` | Destino de doação. |
| `purchase` | Metadados de compra/assinatura. |
| `descriptions` | Catálogo de localização no mesmo diretório. |
| `description_tag` | Chave de localização da descrição curta. |
| `long-description_tag` | Chave de localização da descrição longa. |

Os campos de overlay do AppStream estão intencionalmente fora do escopo
deste documento.
