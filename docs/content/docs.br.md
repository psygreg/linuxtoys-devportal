# Documentação para Desenvolvedores do LinuxToys

O LinuxToys é um aplicativo para Linux projetado para simplificar a instalação, configuração e gerenciamento de softwares e recursos do sistema para usuários de uma ampla variedade de distribuições Linux.

Para desenvolvedores, o LinuxToys fornece uma camada de distribuição capaz de cuidar de grande parte do trabalho específico do Linux normalmente necessário para entregar software aos usuários: compatibilidade com distribuições e hardware, formatos de pacotes, dependências, procedimentos de instalação, atualizações, integração com o sistema e remoção.

Você **não** precisa aprender toda a interface de desenvolvimento do LinuxToys para distribuir um aplicativo através dele.

O LinuxToys oferece suporte desde pacotes simples de aplicativos até recursos complexos do sistema, portanto suas APIs para desenvolvedores incluem ferramentas para diversas situações especializadas e peculiaridades. A maioria dos aplicativos precisa apenas de uma pequena parte desses recursos.

O primeiro passo é escolher o método de integração adequado.

---

<a id="quick-start"></a>

## Início Rápido

As listas de repositório do LinuxToys oferecem suporte a diversos métodos de distribuição, portanto, se quiser economizar tempo, você pode ir direto ao que precisa. Geralmente, não é necessário aprender todos eles — escolha aquele que corresponde à forma como seu aplicativo já é empacotado ou disponibilizado pelo upstream.

### Qual Devo Usar?

| Seu fluxo de lançamento atual                                             | Método recomendado    |
| ------------------------------------------------------------------------- | --------------------- |
| Disponível nos repositórios das distribuições                             | `native`              |
| Publicado no Flathub                                                      | `flathub`             |
| Disponível como um pacote padrão em um repositório do GitHub              | `git`                 |
| Os Lançamentos do GitHub fornecem um único executável                     | `bin`                 |
| Você hospeda um único executável em uma URL direta                        | `url` → `bin`         |
| Você hospeda pacotes `.deb`, `.rpm`, AppImage ou similares                | `url`                 |
| Você distribui um arquivo compactado pré-compilado com múltiplos arquivos | `tar` / `url` → `tar` |

Sempre que possível, **siga o fluxo de lançamento upstream já existente do aplicativo**. Os diferentes métodos das listas de repositório do LinuxToys existem para se adaptar à forma como o software já é distribuído, em vez de exigir que os desenvolvedores criem um formato de pacote específico para o LinuxToys.

Após identificar o método apropriado, procure abaixo pelo atalho correspondente para acessar seu esquema completo, opções de compatibilidade, hooks, traduções e recursos avançados. Se desejar uma página de aplicativo mais completa, com uma descrição mais longa, capturas de tela do aplicativo em ação e botão de doação ou compra, veja também [como configurar uma página de aplicativo do LinuxToys](repositorylists.pt-BR.html#app-pages).

### Pacotes Nativos

**Escolha esta opção se o seu aplicativo estiver disponível através dos repositórios das distribuições Linux.**

O tipo `native` permite que o LinuxToys instale o pacote apropriado utilizando o gerenciador de pacotes da distribuição do sistema. Nomes de pacotes diferentes podem ser definidos para cada distribuição quando houver diferenças entre os ecossistemas.

**Mais indicado para:** aplicativos já empacotados para distribuições como Arch Linux, Fedora, Debian, Ubuntu, openSUSE e suas derivadas.

**Tipo da lista de repositório:** `native`

[Saiba mais sobre pacotes nativos](repositorylists.pt-BR.html#native-package)

### Flathub

**Escolha esta opção se o seu aplicativo estiver publicado no Flathub.**

Se você já distribui seu aplicativo como Flatpak através do Flathub, o LinuxToys pode instalá-lo diretamente utilizando seu ID de aplicativo Flatpak.

Essa geralmente é a opção mais simples para aplicativos cujo principal método de distribuição no Linux é o Flathub.

**Mais indicado para:** aplicativos distribuídos oficialmente através do Flathub.

**Tipo da lista de repositório:** `flathub`

[Saiba mais sobre aplicativos do Flathub](repositorylists.pt-BR.html#flathub-package)

### Lançamentos do Git

**Escolha esta opção se o seu aplicativo publica pacotes Linux prontos para uso através dos Lançamentos do GitHub.**

O tipo `git` permite que o LinuxToys examine o lançamento estável mais recente do repositório upstream, selecione automaticamente o pacote apropriado para o sistema do usuário, faça seu download e o instale.

Isso é útil quando você já publica pacotes Linux instaláveis como arquivos de lançamento e deseja que o LinuxToys acompanhe automaticamente os lançamentos mais recentes, sem precisar manter uma URL de download fixa para cada nova versão.

**Mais indicado para:** projetos que publicam pacotes prontos para uso, como `.deb`, `.rpm`, AppImage, pacotes Flatpak ou outros formatos de pacote compatíveis através dos Lançamentos do GitHub.

**Tipo da lista de repositório:** `git`

[Saiba mais sobre instalações através de lançamentos do Git](repositorylists.pt-BR.html#git-package)

### Binários Independentes

**Escolha esta opção se os seus lançamentos fornecerem um único executável pronto para uso.**

O LinuxToys pode baixar um binário independente, instalá-lo no diretório local de aplicativos do LinuxToys do usuário, torná-lo executável e criar automaticamente seu atalho no menu de aplicativos.

Para Lançamentos do GitHub, utilize o tipo `bin` e forneça o nome exato do arquivo do lançamento. Se o nome do arquivo contiver a versão do lançamento, `$APP_GIT_VERSION` pode ser utilizado como marcador.

Binários independentes disponíveis através de uma URL direta e estável podem ser fornecidos utilizando a opção `bin` dentro de `urls`.

**Mais indicado para:** aplicativos autocontidos distribuídos como um único executável, incluindo binários Linux sem extensão.

**Tipo da lista de repositório:** `bin` para Lançamentos do GitHub, ou `url` com uma URL `bin`.

[Saiba mais sobre aplicativos de binário único](repositorylists.pt-BR.html#single-binary)

### URLs Diretas

**Escolha esta opção se você publica pacotes prontos para uso fora dos repositórios das distribuições.**

O tipo `url` permite que o LinuxToys baixe arquivos instaláveis diretamente de URLs upstream. Isso é útil quando você mesmo publica os pacotes em vez de depender dos repositórios da distribuição do sistema, como normalmente ocorre com software proprietário.

As entradas de URL também podem considerar diferentes distribuições ou sistemas quando downloads diferentes forem necessários.

**Mais indicado para:** pacotes `.deb`, `.rpm`, AppImage, pacotes Flatpak e outros formatos de pacote compatíveis hospedados pelo upstream.

**Tipo da lista de repositório:** `url`

[Saiba mais sobre instalações através de URLs](repositorylists.pt-BR.html#url-fetching)

### Tarballs

**Escolha esta opção se o seu aplicativo for distribuído como um arquivo compactado pré-compilado contendo múltiplos arquivos.**

O LinuxToys pode baixar e extrair tarballs de aplicativos, mantendo seu conteúdo agrupado no diretório local de aplicativos do LinuxToys do usuário.

Como o conteúdo e as etapas de configuração necessárias para arquivos compactados variam entre projetos, instalações por tarball exigem um script de pós-instalação para concluir a configuração do aplicativo.

**Mais indicado para:** aplicativos portáteis pré-compilados distribuídos como `.tar.gz`, `.tar.xz` ou arquivos compactados semelhantes, em vez de um único executável.

**Tipo da lista de repositório:** `tar`, ou `url` com uma URL `tar`.

[Saiba mais sobre instalações por tarball](repositorylists.pt-BR.html#tarball-package)

---

<a id="multi-distro"></a>

## O que o LinuxToys oferece

A distribuição de software para Linux pode rapidamente se tornar complicada quando um aplicativo precisa oferecer suporte a várias distribuições, gerenciadores de pacotes, ambientes de desktop ou configurações de hardware.

O LinuxToys fornece uma infraestrutura comum para lidar com essas diferenças, enquanto apresenta aos usuários uma experiência de instalação consistente.

Dependendo da integração, o LinuxToys pode cuidar de tarefas como:

* detectar a distribuição e as características do sistema do usuário;
* restringir recursos a sistemas compatíveis;
* selecionar os pacotes adequados para diferentes distribuições;
* instalar pacotes nativos, Flatpaks, AppImages e outros formatos de distribuição;
* baixar pacotes diretamente de lançamentos do projeto upstream;
* instalar dependências;
* executar procedimentos de pré e pós-instalação;
* integrar aplicativos à interface do LinuxToys;
* registrar alterações realizadas durante a instalação;
* reverter instalações e alterações de sistema compatíveis;
* gerenciar serviços e outros componentes do sistema;
* lidar com diferenças específicas entre os gerenciadores de pacotes das distribuições;
* oferecer suporte a recursos específicos de determinado hardware ou ambiente de desktop.

Muitos desses recursos existem porque o LinuxToys também distribui drivers, utilitários de sistema, camadas de compatibilidade, recursos de desempenho e outros softwares que exigem uma integração com o sistema consideravelmente maior do que um aplicativo de desktop convencional.

**Um pacote de aplicativo comum geralmente precisará apenas de uma fração deles.**

---

## Escolhendo um método de integração

O LinuxToys oferece duas formas principais para desenvolvedores integrarem seus softwares.

### Listas de Repositórios

**Comece por aqui para a maioria dos aplicativos.**

As Listas de Repositórios fornecem uma maneira declarativa de descrever softwares que o LinuxToys pode instalar. Em vez de escrever um script de instalação, você descreve o aplicativo, sua origem, qual pacote deve ser instalado e quaisquer requisitos de compatibilidade relevantes.

Elas são adequadas para aplicativos que já são distribuídos através de meios convencionais, incluindo aplicativos com:

* pacotes nativos para uma ou mais distribuições Linux;
* versões em Flatpak;
* versões em AppImage;
* pacotes publicados através dos sistemas de lançamentos do projeto upstream;
* pacotes hospedados diretamente pelo desenvolvedor;
* dependências simples;
* requisitos de compatibilidade simples;
* pequenas etapas de pré ou pós-instalação.

As Listas de Repositórios permitem que o LinuxToys realize a instalação utilizando sua infraestrutura existente, em vez de exigir que cada desenvolvedor implemente o gerenciamento de pacotes e a detecção do sistema de forma independente.

Elas também podem expressar condições mais avançadas quando necessário, incluindo distribuição, arquitetura, CPU, GPU, ambiente de desktop e outros requisitos de compatibilidade.

Para muitos aplicativos, uma entrada em uma Lista de Repositórios pode ser **o único código específico do LinuxToys que você precisará manter**.

### Biblioteca Principal e Scripts Completos

Use um script completo do LinuxToys quando seu software exigir um procedimento que não possa ser adequadamente descrito como uma instalação de pacote.

A Biblioteca Principal do LinuxToys fornece funções Bash reutilizáveis e informações sobre o sistema para essas integrações mais complexas.

Essa abordagem é adequada para recursos que precisam realizar operações como:

* procedimentos complexos de instalação ou migração;
* configurações extensivas do sistema;
* operações condicionais de acordo com o sistema hospedeiro;
* alterações no sistema de arquivos que precisam ser registradas para remoção;
* gerenciamento de serviços systemd;
* alterações no carregador de inicialização ou na configuração de boot;
* procedimentos incomuns de gerenciamento de pacotes;
* múltiplas etapas de instalação dependentes entre si;
* chamada de outros recursos do LinuxToys;
* comportamentos personalizados de reversão ou limpeza;
* tratamento especializado para peculiaridades de distribuições ou hardware.

A Biblioteca Principal existe para que esses scripts possam utilizar a mesma infraestrutura de detecção de compatibilidade, abstração do gerenciamento de pacotes, registro de transações e integração com o sistema utilizada pelo próprio LinuxToys.

Um script completo, portanto, é consideravelmente mais poderoso do que uma Lista de Repositórios — mas a maioria dos desenvolvedores de aplicativos **não precisa dessa complexidade adicional**.

---

<a id="official-support"></a>

## Oferecendo suporte oficial

Desenvolvedores também podem entrar em contato com o projeto LinuxToys para solicitar o status de **suporte oficial** para seus aplicativos.

Aplicativos oficialmente suportados são identificados como tal dentro do LinuxToys, diferenciando integrações mantidas em colaboração com seus desenvolvedores upstream. Esse status também cria um canal mais direto entre os usuários do LinuxToys e o projeto responsável pelo aplicativo.

Entre as vantagens do suporte oficial estão:

* identificação visual do aplicativo como oficialmente suportado no LinuxToys;
* maior destaque e facilidade de identificação pelos usuários;
* encaminhamento mais eficiente de problemas específicos do aplicativo ao desenvolvedor upstream;
* possibilidade de receber informações relevantes coletadas pelo sistema de relatórios de bugs do LinuxToys através do GitHub, facilitando a investigação de problemas encontrados pelos usuários;
* colaboração mais próxima com o projeto LinuxToys para manter e aprimorar a integração ao longo do tempo.

O suporte oficial **não exige que o aplicativo utilize todos os recursos de integração do LinuxToys**. Um aplicativo distribuído através de uma Lista de Repositórios simples pode receber suporte oficial da mesma forma que uma integração mais complexa.

Caso tenha interesse em oferecer suporte oficial ao seu aplicativo através do LinuxToys, informe os mantenedores do projeto em sua *pull request*.

---

## Instalar com o LinuxToys

O LinuxToys fornece um esquema de URI personalizado que permite que sites solicitem a instalação de softwares disponíveis através do LinuxToys. Isso possibilita que desenvolvedores upstream disponibilizem um botão **Instalar com o LinuxToys** diretamente em seus sites.

A URI segue este formato:

```text
linuxtoys://install/<nome>
```

Por exemplo, uma entrada de lista de repositórios chamada `Hardinfo2` pode ser aberta com:

```text
linuxtoys://install/Hardinfo2
```

Nomes que contêm espaços devem utilizar a codificação percentual padrão de URIs. Por exemplo:

```text
linuxtoys://install/Amethyst%20Mod%20Manager
```

Abrir uma URI de instalação **não** instala imediatamente o software solicitado. O LinuxToys será aberto e apresentará o aplicativo solicitado ao usuário para confirmação antes de prosseguir.

### Adicionando um botão "Instalar com o LinuxToys"

Oferecemos uma imagem de botão padrão pronta para uso, por praticidade. A versão em inglês já pronta do botão pode ser obtida salvando a imagem abaixo. Utilizamos a fonte *Adwaita Sans Bold* para o texto.

![Botão em inglês](/assets/installwithlinuxtoys_en.webp)

Também oferecemos uma versão em branco do botão se quiser colocar o texto em outro idioma ou fonte. Se desejar, obtenha-a salvando a imagem abaixo.

![Botão em branco](/assets/installwithlinuxtoys_base.webp)

Um botão básico pode ser adicionado com:

```html
<a href="linuxtoys://install/Hardinfo2">
    <img
        src="/assets/installwithlinuxtoys-en.webp"
        alt="Instalar com o LinuxToys"
    >
</a>
```

Dessa forma, a imagem do botão funciona como um link que inicia o LinuxToys.

Você não **precisa** usar o botão padrão. O formato URI do LinuxToys pode ser utilizado em um botão ou link a seu critério.

### Fornecendo uma alternativa para usuários sem o LinuxToys

Os navegadores não fornecem uma maneira padronizada para que um link comum determine se um manipulador de URI personalizado está instalado. Se a página na qual o botão está sendo utilizado permitir JavaScript, uma pequena rotina alternativa pode redirecionar o visitante para o site do LinuxToys caso a URI não possa ser aberta:

```html
<a href="linuxtoys://install/Hardinfo2"
   onclick="installWithLinuxToys(event, 'Hardinfo2')">
    <img
        src="/assets/installwithlinuxtoys-en.webp"
        alt="Instalar com o LinuxToys"
    >
</a>

<script>
function installWithLinuxToys(event, name) {
    event.preventDefault();

    const uri = `linuxtoys://install/${encodeURIComponent(name)}`;
    let pageHidden = false;

    const onVisibilityChange = () => {
        if (document.hidden) {
            pageHidden = true;
        }
    };

    document.addEventListener("visibilitychange", onVisibilityChange);

    window.location.href = uri;

    setTimeout(() => {
        document.removeEventListener("visibilitychange", onVisibilityChange);

        if (!pageHidden) {
            window.location.href = "https://linux.toys/";
        }
    }, 1500);
}
</script>
```

Para utilizar o botão com outra entrada de lista de repositórios do LinuxToys, substitua `Hardinfo2` pelo seu `name`:

```html
<a href="linuxtoys://install/Amethyst%20Mod%20Manager"
   onclick="installWithLinuxToys(event, 'Amethyst Mod Manager')">
    <img
        src="/assets/installwithlinuxtoys-en.webp"
        alt="Instalar com o LinuxToys"
    >
</a>
```

`encodeURIComponent()` cuida da codificação de espaços e outros caracteres que possuem significado especial em URIs presentes no nome da entrada da lista de repositórios.

> **Nota:** alguns renderizadores de Markdown, incluindo plataformas que sanitizam HTML incorporado, podem remover elementos `<script>` ou JavaScript inline. Nessas plataformas, utilize apenas o link `linuxtoys://` básico ou implemente a rotina alternativa no JavaScript do site que hospeda a documentação renderizada.

---

## Qual documentação devo ler?

A maneira mais fácil de decidir é começar pelo que você pretende distribuir.

| Seu aplicativo ou recurso...                                                          | Comece por                 |
| ------------------------------------------------------------------------------------- | -------------------------- |
| Já está disponível como pacotes Linux convencionais                                   | **Listas de Repositórios** |
| É distribuído como AppImage ou Flatpak                                                | **Listas de Repositórios** |
| Publica pacotes instaláveis através de lançamentos ou de um servidor de downloads     | **Listas de Repositórios** |
| Precisa instalar dependências antes do pacote principal                               | **Listas de Repositórios** |
| Precisa executar comandos simples antes ou depois da instalação                       | **Listas de Repositórios** |
| Possui pacotes diferentes para diferentes distribuições                               | **Listas de Repositórios** |
| Deve aparecer apenas em determinados hardwares, distribuições ou ambientes de desktop | **Listas de Repositórios** |
| Exige um procedimento de instalação personalizado e complexo                          | **Biblioteca Principal**   |
| Realiza alterações extensivas no sistema operacional                                  | **Biblioteca Principal**   |
| Precisa de controle detalhado de transações e reversão                                | **Biblioteca Principal**   |
| Gerencia serviços, configurações de boot ou outros componentes do sistema             | **Biblioteca Principal**   |
| Não pode ser representado adequadamente como um pacote com hooks opcionais            | **Biblioteca Principal**   |

<a id="declarative-deployment"></a>

### → [Documentação das Listas de Repositórios](repositorylists.html)

Aprenda a descrever um aplicativo de forma declarativa, definir seus pacotes e dependências, especificar requisitos de compatibilidade, fornecer metadados e ícones, utilizar os métodos de distribuição compatíveis e adicionar hooks opcionais de instalação.

**Este é o ponto de partida recomendado para desenvolvedores de aplicativos.**

### → [Documentação da Biblioteca Principal](corelibraries.html)

Aprenda a criar scripts completos para o LinuxToys utilizando seus recursos de compatibilidade, gerenciamento de pacotes, sistema de arquivos, systemd, boot, informações do sistema e gerenciamento de transações.

**Utilize esta opção quando uma Lista de Repositórios não for suficiente para sua integração.**

> Os nomes dos arquivos acima podem ser ajustados para corresponder à estrutura final da documentação.

---

## Você provavelmente não precisa de tudo

Os dois conjuntos de documentação descrevem recursos utilizados em diferentes partes do próprio LinuxToys.

Isso significa que você encontrará opções que podem ter pouca ou nenhuma relação com o seu aplicativo.

Por exemplo, o LinuxToys precisa oferecer suporte a softwares que vão desde aplicativos de desktop convencionais até ferramentas para GPUs, drivers, modificações em carregadores de inicialização, componentes de compatibilidade e configurações de baixo nível do sistema. A infraestrutura necessária para esses recursos é disponibilizada para que desenvolvedores possam resolver problemas semelhantes quando necessário.

Isso **não** é uma lista de coisas que toda integração deve implementar.

Um desenvolvedor distribuindo um aplicativo convencional pode precisar fornecer apenas:

1. os metadados do aplicativo;
2. o repositório upstream ou a origem do pacote;
3. o nome do pacote ou formato de distribuição apropriado; e
4. quaisquer restrições de compatibilidade que realmente se apliquem.

Todo o resto pode ser ignorado até que seu aplicativo realmente precise desses recursos.

O mesmo princípio se aplica a scripts completos: utilize as funções da Biblioteca Principal que resolvam o problema em questão, em vez de tentar incorporar todos os recursos fornecidos pelo LinuxToys.

---

## Prefira a integração mais simples

Quando várias abordagens puderem alcançar o mesmo resultado, prefira aquela que delega mais trabalho ao LinuxToys.

Uma Lista de Repositórios declarativa geralmente é preferível a reproduzir o mesmo procedimento de instalação em um script personalizado. Ela é mais fácil de revisar, mais fácil de manter e permite que melhorias na infraestrutura de gerenciamento de pacotes e compatibilidade do LinuxToys beneficiem seu aplicativo automaticamente.

Da mesma forma, quando um script completo for necessário, prefira as funções da Biblioteca Principal do LinuxToys em vez de implementar diretamente comandos específicos de cada distribuição sempre que já existir uma abstração adequada.

Isso mantém as integrações consistentes com o restante do LinuxToys e reduz a quantidade de comportamento específico de cada distribuição que os desenvolvedores precisam manter por conta própria.

---

## Por onde começar

Para a maioria dos desenvolvedores:

**Aplicativo → Lista de Repositórios → LinuxToys cuida da instalação**

Comece pela **[Documentação das Listas de Repositórios](repositorylists.html)** e implemente apenas os campos relevantes para o seu aplicativo.

Para integrações complexas em nível de sistema:

**Recurso → Script do LinuxToys → Biblioteca Principal → Sistema**

Comece pela **[Documentação da Biblioteca Principal](corelibraries.html)** e utilize as bibliotecas relevantes para as operações que seu recurso precisa realizar.

Você sempre poderá migrar para uma integração mais avançada posteriormente caso os requisitos do seu software aumentem.

O objetivo não é fazer com que os desenvolvedores aprendam todos os recursos internos do LinuxToys. O objetivo é fornecer infraestrutura suficiente para que os desenvolvedores precisem implementar apenas as partes que realmente são específicas de seus softwares.
