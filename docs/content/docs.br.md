# Documentação do LinuxToys para Desenvolvedores

O LinuxToys oferece caminhos diferentes de integração. Para a maioria dos aplicativos desktop, o ponto de partida correto é o **AppStream**.

> **Se o seu aplicativo está disponível pelo Flathub ou pelos repositórios nativos de uma distribuição compatível, este é o único guia de integração que você precisa.**

O LinuxToys descobre esses aplicativos pelo AppStream e transforma os metadados existentes em páginas de aplicativo e entradas de instalação. Não é necessário criar uma definição separada apenas para disponibilizar o mesmo aplicativo no LinuxToys.

## Escolha seu caminho de integração

### Seu aplicativo está disponível pelo AppStream

Continue nesta página se ele está no **Flathub** ou em um repositório de distribuição compatível e possui metadados AppStream utilizáveis. Metadados específicos do LinuxToys só são necessários quando você quiser estender a entrada existente com um **overlay AppStream**.

### Seu aplicativo é distribuído de forma independente

Use as [Listas de Repositório](repositorylists.pt-BR.html) quando o LinuxToys precisar de uma definição própria de instalação, como para releases Git, AppImages, URLs diretas, tarballs, binários independentes ou código-fonte que precise ser compilado.

### Você está integrando um recurso do LinuxToys ou do sistema

Use a [Biblioteca Shell](corelibraries.pt-BR.html) para recursos de sistema e procedimentos personalizados que pertençam ao próprio LinuxToys, em vez de descrever um aplicativo distribuível.

---

## Aplicativos AppStream

O AppStream é o caminho preferencial para aplicativos já distribuídos por fontes de software Linux estabelecidas. O LinuxToys consome catálogos dos repositórios nativos compatíveis e do Flathub, adapta os componentes ao seu catálogo e aplica suas regras de compatibilidade e seleção de fonte.

### O que o LinuxToys obtém do AppStream

Bons metadados upstream podem fornecer nome e resumo, ícone, desenvolvedor, licença, links do projeto, descrição longa, capturas de tela, categorias, tipo do aplicativo, informações de releases e outros metadados AppStream.

Por isso, o primeiro passo para uma boa apresentação no LinuxToys é **manter os metadados AppStream upstream completos e corretos**.

### Categorias

O LinuxToys mapeia as categorias padrão do AppStream/Desktop Menu para o próprio catálogo, considerando tanto categorias amplas quanto categorias adicionais mais específicas. Descreva o aplicativo corretamente usando o sistema normal do AppStream; não é necessário adicionar categorias específicas do LinuxToys aos metadados upstream.

### Pacotes nativos e Flathub

O mesmo aplicativo pode existir em mais de uma fonte. O LinuxToys pode combinar essas representações em uma única experiência e, quando aplicável, oferecer um seletor de fonte.

**Não** crie uma Lista de Repositório apenas porque o aplicativo existe tanto nativamente quanto no Flathub.

### Páginas de aplicativo

Aplicativos AppStream são apresentados em páginas que podem incluir descrição, capturas de tela, desenvolvedor, licença, links, avaliações, fonte de instalação, controles de instalação e a ação **Abrir** quando o aplicativo instalado puder ser iniciado.

![página de aplicativo](/assets/app-page-br.webp)

O LinuxToys também acompanha o estado de instalação e integra essas instalações à fila e ao fluxo de remoção.

---

## Overlays AppStream

A maioria dos aplicativos AppStream não precisa de definição específica do LinuxToys.

Um **overlay AppStream** adiciona informações ou comportamento revisado sobre um componente AppStream existente. Ele **não cria um novo aplicativo** nem substitui sua fonte normal de instalação.

```json
{
  "appstream-name": "org.example.App"
}
```

Use o ID de componente AppStream. O `.desktop` final é normalizado e não é necessário.

Overlays aceitam apenas:

```text
appstream-name
purchase
dependencies
overrides
```

### Dependências

Dependência nativa:

```json
{
  "appstream-name": "org.example.App",
  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": "example-helper",
        "ubuntu": "example-helper",
        "fedora": "example-helper",
        "arch": "example-helper"
      }
    }
  ]
}
```

Dependência do Flathub:

```json
{
  "appstream-name": "org.example.App",
  "dependencies": [
    {
      "type": "flathub",
      "package-name": "org.example.Runtime"
    }
  ]
}
```

As dependências complementam a instalação AppStream; não substituem a fonte nativa ou Flatpak selecionada.

### Hooks pré e pós-instalação

```json
{
  "appstream-name": "org.example.App",
  "overrides": {
    "pre": {
      "script": "example/pre-install.sh"
    },
    "post": {
      "script": "example/post-install.sh"
    }
  }
}
```

Os scripts são recursos locais do repositório. Mantenha os hooks pequenos e use-os apenas quando o comportamento não puder ser representado declarativamente.

### Overrides de permissões Flatpak

```json
{
  "appstream-name": "org.example.App",
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
}
```

Os escopos são `user` e `system`. Tipos compatíveis:

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

Overlays AppStream **não** aceitam controles de seleção de fonte como `skip-user`; fonte e escopo continuam sob responsabilidade do fluxo AppStream.

### Compras e assinaturas

Compra única:

```json
{
  "appstream-name": "org.example.App",
  "purchase": {
    "url": "https://example.com/buy",
    "price": 49.99
  }
}
```

Assinatura:

```json
{
  "appstream-name": "org.example.App",
  "purchase": {
    "url": "https://example.com/subscribe",
    "sub_price": 9.99
  }
}
```

As duas podem ser combinadas, e os formatos em níveis das Listas de Repositório também são aceitos quando necessários. Os links apontam para o destino do próprio desenvolvedor ou empresa; o LinuxToys não processa o pagamento.

### Combinando recursos

```json
{
  "appstream-name": "org.example.App",
  "purchase": {
    "url": "https://example.com/pricing",
    "price": 49.99
  },
  "dependencies": [
    {
      "type": "native",
      "package-name": {
        "debian": "example-helper",
        "fedora": "example-helper",
        "arch": "example-helper"
      }
    }
  ],
  "overrides": {
    "post": {
      "script": "example/post-install.sh"
    }
  }
}
```

Comece sem overlay. Adicione um apenas quando a entrada AppStream precisar de comportamento específico do LinuxToys.

---

## Links de instalação do LinuxToys

O LinuxToys oferece uma URI para que sites, documentações e outros aplicativos enviem uma solicitação de instalação diretamente a ele.

Para AppStream, use o ID estável do componente:

```text
linuxtoys://install/org.example.App
```

```html
<a href="linuxtoys://install/org.example.App">
  Instalar com LinuxToys
</a>
```

O LinuxToys resolve o alvo entre os aplicativos disponíveis e compatíveis no sistema e continua pelo fluxo normal de instalação. Links também podem apontar para entradas curadas do LinuxToys, mas o ID AppStream é preferível para aplicativos AppStream.

### Botão "Instalar com LinuxToys"

#### Botão pronto em inglês

<img src="/assets/installwithlinuxtoys_en.webp"
     alt="Install with LinuxToys"
     data-no-theme-image>

#### Botão em branco para edição

<img src="/assets/installwithlinuxtoys_base.webp"
     alt="Install with LinuxToys"
     data-no-theme-image>

```html
<a href="linuxtoys://install/org.example.App">
  <img src="install-with-linuxtoys.svg" alt="Instalar com LinuxToys">
</a>
```

Substitua `org.example.App` pelo ID AppStream real.

---

## Aplicativos fora do AppStream

Se o LinuxToys não puder obter e instalar o aplicativo por uma fonte AppStream adequada, use uma **Lista de Repositório**.

Elas descrevem releases Git, AppImages, URLs diretas, tarballs, binários independentes, builds a partir do código-fonte e outros métodos compatíveis, além de regras de compatibilidade, dependências, hooks, serviços e metadados de página quando necessário.

Continue pela [documentação de Listas de Repositório](repositorylists.pt-BR.html).

---

## Integrações de sistema e recursos do LinuxToys

Para ajustes do sistema, fluxos de drivers, manutenção, integrações de plataforma ou outros recursos cuja lógica pertença diretamente ao LinuxToys, use a **Biblioteca Shell** e o modelo de integração por scripts.

Continue pela [documentação da Biblioteca Shell](corelibraries.pt-BR.html).

A divisão é intencional:

- **AppStream**: aplicativos distribuídos por fontes de software compatíveis.
- **Listas de Repositório**: aplicativos que o LinuxToys precisa obter ou instalar por outro método compatível.
- **Biblioteca Shell**: procedimentos nativos do LinuxToys e integrações de sistema.
