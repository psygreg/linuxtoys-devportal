# API de Scripts do LinuxToys

O LinuxToys oferece suporte a scripts Bash completos para softwares e recursos que exigem mais controle do que uma [lista de repositório](repositorylists.html). Esses scripts podem aproveitar a detecção de compatibilidade do LinuxToys, a abstração do gerenciamento de pacotes, o rastreamento de transações do sistema de arquivos, os auxiliares do systemd, o gerenciamento do carregador de inicialização, os recursos de interação com o usuário e o sistema de reversão automática.

Este documento aborda a interface de scripts para scripts `.sh` comuns do LinuxToys.

As listas de repositórios são um recurso separado, com documentação própria, e intencionalmente não são abordadas aqui.

---

## Estrutura Básica de um Script

Um script do LinuxToys é um script Bash comum com um cabeçalho de metadados no início:

```bash
# name: Example Application
# version: 1.0
# description: Installs and configures Example Application.
# icon: example.svg
# compat: debian, ubuntu, fedora, arch
# reboot: no
# revert: yes

pkg_install example-package

info "$finishmsg"
```

O LinuxToys lê os metadados apenas do bloco inicial de comentários. Assim que o parser encontra a primeira linha que não começa com `#`, a leitura do cabeçalho é encerrada.

Por esse motivo, todos os cabeçalhos do LinuxToys devem aparecer antes da parte executável do script.

Em geral, **você não precisa carregar (`source`) as bibliotecas do LinuxToys por conta própria**. Quando o LinuxToys inicia um script, seu carregador de bibliotecas examina o script, determina quais bibliotecas modulares são necessárias, exporta as flags de biblioteca apropriadas e carrega a biblioteca principal `linuxtoys.bash` antes de carregar o script propriamente dito.

A execução resultante é conceitualmente semelhante a:

```bash
export SCRIPT_DIR="/path/to/linuxtoys"
export FS_OPS=1
export PACKAGE_OPS=1
# ...

source "$SCRIPT_DIR/libs/linuxtoys.bash"
source "/path/to/script.sh"
```

O preâmbulo real é construído automaticamente pelo `library_loader.py`.

---

## Cabeçalhos de Metadados de Exibição

### `name`

```bash
# name: Example Application
```

Define o nome do script exibido ao usuário.

Esse nome é exibido no LinuxToys e também é usado quando um script filho chamado por `call_script` é registrado como uma transação.

Se nenhum nome for fornecido, scripts comuns usam `No Name` por padrão; scripts locais do usuário podem usar o nome do arquivo como alternativa.

Para scripts distribuídos com o LinuxToys, sempre forneça um nome.

---

### `version`

```bash
# version: 1.4
```

Fornece metadados de versão para o script.

O parser normal usa `N/A` como valor padrão para esse campo.

Esta é a versão da definição do script/recurso do LinuxToys, não necessariamente a versão do software que ele instala.

---

### `description`

```bash
# description: Installs the Example graphics configuration utility.
```

Fornece a descrição exibida ao usuário pelo LinuxToys.

`name` e `description` também podem utilizar a busca de traduções do LinuxToys quando traduções são fornecidas ao parser.

---

### `icon`

```bash
# icon: example.svg
```

Especifica o ícone associado ao recurso.

Sem um, o parser usa como padrão:

```text
application-x-executable
```

---

### `repo`

```bash
# repo: https://github.com/example/example
```

Associa um repositório upstream ao recurso.

Se omitido, esse campo fica vazio.

---

## Compatibilidade com Sistemas Operacionais

### `compat`

```bash
# compat: debian, ubuntu, fedora
```

Restringe um script a sistemas que correspondam a pelo menos uma das chaves de compatibilidade fornecidas.

Múltiplas chaves são separadas por vírgulas.

Por exemplo:

```bash
# compat: arch, cachy, manjaro
```

significa:

> Show/run this script if the machine matches Arch, CachyOS, or Manjaro.

O cabeçalho também oferece suporte a exclusões com `!`:

```bash
# compat: debian, !deepin
```

O LinuxToys separa as entradas fornecidas em conjuntos de inclusão e exclusão. Se houver chaves de inclusão, pelo menos uma delas deve corresponder. Qualquer chave de exclusão correspondente torna o script incompatível.

Portanto, uma regra contendo apenas uma lista de exclusão também é válida:

```bash
# compat: !ubuntu
```

Isso permite o script em todos os sistemas compatíveis com o LinuxToys, exceto Ubuntu.

### Chaves úteis de sistema operacional

A detecção de compatibilidade atual inclui famílias e derivados do Linux como:

```text
debian
ubuntu
arch
cachy
manjaro
fedora
rhel
suse
ostree
ublue
solus
zorin
deepin
pika
```

Não faça sua própria análise de `/etc/os-release` apenas para distinguir as famílias comumente suportadas. A biblioteca shell disponibiliza funções correspondentes para ramificações em tempo de execução, conforme descrito posteriormente neste documento.

---

## Compatibilidade de GPU

### `gpu`

Restringe um script de acordo com o hardware gráfico ou capacidade computacional:

```bash
# gpu: amd
```

Os valores especiais suportados são:

```text
amd
intel
nvidia
rocm
xe
```

Eles correspondem às chaves de compatibilidade do LinuxToys:

```text
gpu-amd
gpu-intel
gpu-nvidia
gpu-rocm
gpu-xe
```

Múltiplos alvos de GPU podem ser fornecidos:

```bash
# gpu: amd, intel
```

Qualquer alvo correspondente é suficiente.

### `rocm`

```bash
# gpu: rocm
```

Exige hardware considerado compatível com ROCm pelo LinuxToys.

### `xe`

```bash
# gpu: xe
```

Exige uma GPU Intel considerada compatível com o caminho Intel Compute Runtime/Xe.

A camada de compatibilidade do LinuxToys gera separadamente as chaves `gpu-rocm`, `gpu-xe` e de GPU híbrida quando apropriado.

---

## Compatibilidade de CPU

### `cpu`

Restringe um recurso de acordo com o fabricante da CPU:

```bash
# cpu: amd
```

or:

```bash
# cpu: intel
```

Múltiplos valores podem ser especificados:

```bash
# cpu: amd, intel
```

Os valores específicos reconhecidos correspondem a `cpu-amd` e `cpu-intel`. Outros valores são tratados como exigindo uma chave genérica de CPU.

---

## Compatibilidade com Ambiente de Desktop

### `desktop`

Restringe um script de acordo com o desktop ativo:

```bash
# desktop: gnome
```

Os valores específicos reconhecidos atualmente são:

```text
gnome
plasma
other
```

Múltiplos desktops podem ser permitidos:

```bash
# desktop: gnome, plasma
```

Internamente, eles correspondem a:

```text
desktop-gnome
desktop-plasma
desktop-other
```

Use esse cabeçalho quando um recurso depender fundamentalmente de um ambiente de desktop. Se o script precisar apenas realizar operações ligeiramente diferentes em desktops diferentes, ele pode fazer a ramificação em tempo de execução.

---

## Compatibilidade com systemd

### `systemd`

Exigir systemd:

```bash
# systemd: yes
```

Um cabeçalho vazio tem o mesmo significado:

```bash
# systemd:
```

Exigir, em vez disso, um sistema que não use systemd:

```bash
# systemd: no
```

O LinuxToys também detecta automaticamente algumas dependências implícitas do systemd. Se um script chamar `pkg_flat` ou uma função correspondente a `sysd_*` e não possuir um cabeçalho `systemd` explícito, o LinuxToys o trata como dependente do systemd.

Isso significa que um script como:

```bash
sysd_enable example.service
sysd_start example.service
```

normalmente não precisa adicionar manualmente:

```bash
# systemd: yes
```

embora declarar explicitamente o requisito possa tornar o script mais fácil de compreender.

---

## Compatibilidade com Wayland/X11

### `wayland`

Para um script exclusivo para Wayland:

```bash
# wayland: yes
```

or:

```bash
# wayland: true
```

Para um script exclusivo para X11:

```bash
# wayland: no
```

or:

```bash
# wayland: false
```

Se esse cabeçalho estiver ausente, o LinuxToys considera que o recurso é neutro e pode ser executado em qualquer um dos tipos de sessão.

---

## Compatibilidade com GPU Híbrida

### `hybridgpu`

Para um recurso que deve existir **somente** em sistemas híbridos NVIDIA + Intel/AMD:

```bash
# hybridgpu: only
```

Para impedir que ele apareça em máquinas com GPU híbrida:

```bash
# hybridgpu: no
```

A implementação de compatibilidade também oferece filtragem por chaves de compatibilidade para sistemas híbridos:

```bash
# hybridgpu: gpu-intel
```

ou exclusões:

```bash
# hybridgpu: !gpu-amd
```

A lista de inclusão/exclusão é avaliada somente quando a máquina possui a chave de compatibilidade `hybridgpu`.

Para casos simples, prefira `only`, `no` ou nenhum cabeçalho.

---

## Compatibilidade por ID de Dispositivo

### `deviceids`

Restringe um script a máquinas que contenham IDs PCI ou USB específicos:

```bash
# deviceids: a69c
```

Vários IDs podem ser fornecidos:

```bash
# deviceids: 8086, 7d55, 8086:7d55
```

O LinuxToys normaliza os IDs removendo um `0x` inicial e os compara com IDs de fabricante PCI e USB detectados, IDs de produto/dispositivo e IDs combinados `fabricante:produto`.

Assim, estas são formas equivalentes quando aplicável:

```text
8086
0x8086
```

Para máxima precisão, use um par combinado de fabricante/produto quando disponível:

```bash
# deviceids: 1234:abcd
```

---

## Compatibilidade com Contêineres

### `nocontainer`

Para impedir que um script seja executado dentro de qualquer contêiner:

```bash
# nocontainer
```

or:

```bash
# nocontainer:
```

Para impedi-lo apenas em contêineres baseados em distribuições selecionadas:

```bash
# nocontainer: debian, ubuntu
```

Para disponibilizar um recurso **somente dentro de contêineres**:

```bash
# nocontainer: invert
```

Ele também pode ser limitado a ambientes de contêiner específicos:

```bash
# nocontainer: invert, debian
```

O comportamento suportado é:

```text
# nocontainer
    Host: allowed
    Container: blocked

# nocontainer: fedora
    Host: allowed
    Fedora container: blocked
    Other container: allowed

# nocontainer: invert
    Host: blocked
    Container: allowed

# nocontainer: invert, debian
    Host: blocked
    Debian container: allowed
    Other container: blocked
```

### Proteção automática para Flatpak/AppImage

Você não precisa adicionar `nocontainer` manualmente apenas porque seu script chama:

```bash
pkg_flat ...
pkg_appimage ...
flatpak_in_lib
```

O LinuxToys trata automaticamente scripts que utilizam esses recursos como incompatíveis com contêineres.

Ele também detecta URLs de AppImage/Flatpak estaticamente visíveis passadas por `pkg_fromurl`.

A restrição automática de Flatpak/AppImage tem precedência sobre metadados explícitos de contêiner.

---

## Filtragem por Localização

### `localize`

Um script pode ser restrito a localidades selecionadas:

```bash
# localize: pt
```

or:

```bash
# localize: pt, es
```

Se `localize` estiver ausente, o script é exibido para todas as localidades.

Se presente, a localidade detectada do usuário deve estar na lista separada por vírgulas.

Esse cabeçalho destina-se a funcionalidades que realmente só fazem sentido em determinadas localidades. Ele não é necessário apenas porque o nome ou a descrição precisam de tradução.

---

## Requisito de Reinicialização

### `reboot`

Para um recurso que exige reinicialização após uma instalação bem-sucedida:

```bash
# reboot: yes
```

Para scripts comuns:

```bash
# reboot: no
```

`no` é o padrão.

Para alterações que exigem reinicialização somente em sistemas rpm-ostree/Universal Blue:

```bash
# reboot: ostree
```

O LinuxToys interpreta `ostree` como requisito de reinicialização somente quando o conjunto de compatibilidade atual contém `ostree` ou `ublue`.

---

## Reversão e Rollback Automático

### `revert`

O LinuxToys mantém um mapa de transações para as operações dos scripts e pode usar essas informações para desfazer alterações.

O padrão é:

```bash
# revert: yes
```

Omitir o cabeçalho também resulta em `yes` por padrão.

### Reversão completa

```bash
# revert: yes
```

Permite ambos:

* rollback automático após uma instalação malsucedida;
* remoção manual pelo LinuxToys.

### Sem reversão

```bash
# revert: no
```

Desativa tanto a reversão automática quanto a manual.

Use isso somente quando o LinuxToys não puder determinar com segurança como desfazer a operação.

### Remoção personalizada interna/manual

```bash
# revert: internal
```

Mantém o suporte a rollback automático, mas indica que a remoção manual exige o processo interno do próprio script em vez da remoção normal baseada em transações do LinuxToys.

### Reversão condicional

A reversão pode ser limitada por chaves de compatibilidade:

```bash
# revert: ubuntu, debian
```

ou excluída em alvos específicos:

```bash
# revert: !ostree
```

Os modos suportados e a sintaxe de listas de inclusão/exclusão são definidos pela camada de compatibilidade.

Para a maioria dos scripts LinuxToys bem-comportados, mantenha a reversão ativada.

---

## Exemplo de Cabeçalho

Um cabeçalho razoavelmente completo para um script comum pode ser assim:

```bash
# name: Example GPU Tool
# version: 1.2
# description: Installs and configures Example GPU Tool.
# icon: example-gpu.svg
# repo: https://github.com/example/example-gpu-tool
# compat: fedora, rhel, arch, cachy
# gpu: amd
# cpu: amd, intel
# desktop: gnome, plasma
# systemd: yes
# wayland: yes
# nocontainer
# reboot: no
# revert: yes
```

Não adicione restrições apenas porque elas estão disponíveis. A ausência de um cabeçalho de compatibilidade normalmente significa que aquela dimensão não possui restrições.

Por exemplo, um software que funciona em todas as GPUs deve **não possuir cabeçalho `gpu`**, em vez de:

```bash
# gpu: amd, intel, nvidia
```

---

## Carregamento Automático de Bibliotecas

A biblioteca Bash do LinuxToys é dividida em vários módulos.

A biblioteca principal sempre carrega:

```text
sysinfo.bash
```

Outros módulos são carregados condicionalmente de acordo com flags geradas automaticamente:

```text
FS_OPS       -> fsops.bash
PACKAGE_OPS  -> packages.bash
BOOT_OPS     -> boot.bash
MISC_OPS     -> misc.bash
SYSD_OPS     -> sysd.bash
```

O carregador examina estaticamente o script em busca de funções pertencentes a esses módulos. Ele também resolve dependências transitivas entre bibliotecas.

Portanto, um script pode simplesmente conter:

```bash
pkg_install curl
prep_create /etc/example.conf
```

sem escrever manualmente:

```bash
source "$SCRIPT_DIR/libs/packages.bash"
source "$SCRIPT_DIR/libs/fsops.bash"
```

### Chamadas de função geradas dinamicamente

O scanner é deliberadamente conservador, mas não é um parser Bash completo. Se o nome de uma função for gerado dinamicamente, o carregador pode não conseguir inferi-lo.

Por exemplo:

```bash
operation="pkg_install"
"$operation" example
```

Para esses casos incomuns, o script pode definir explicitamente a flag correspondente antes de carregar `linuxtoys.bash` quando for executado fora do caminho normal do carregador. A documentação do carregador identifica explicitamente essa alternativa para nomes de função calculados.

Para scripts LinuxToys comuns, chamadas diretas de função são fortemente recomendadas.

---

## Funções de Erro e Mensagens

A biblioteca principal fornece funções de mensagens adaptadas à interface.

### `info`

```bash
info "Installation completed."
```

Exibe uma caixa de diálogo informativa quando a interação gráfica está disponível e, caso contrário, utiliza a saída do terminal.

Portanto, a mensagem normal de conclusão pode ser escrita como:

```bash
info "$finishmsg"
```

---

### `warn`

```bash
warn "This feature requires manual configuration."
```

Exibe um aviso.

---

### `error`

```bash
error "Optional component could not be configured."
```

Relata um erro, mas devolve o controle ao script.

Use isso para uma falha que **não** deve encerrar toda a instalação.

---

### `die`

```bash
die "Failed to install Example."
```

Relata um erro fatal e encerra o script com status de saída `1`.

Uso típico:

```bash
some_command || die "some_command failed"
```

Aliases legados continuam disponíveis:

```text
fatal      -> die
nonfatal   -> error
zeninf     -> info
zenwrn     -> warn
zenask     -> question
sudo_rq    -> askpass sudo
zenpass    -> askpass password
```

Código novo geralmente deve usar os nomes mais recentes, exceto quando uma convenção existente do LinuxToys tornar o alias legado mais claro.

---

## Fazendo uma Pergunta ao Usuário

### `question`

```bash
if question "Example Installer" \
    "Would you like to enable the optional component?"; then
    ...
fi
```

A assinatura é:

```bash
question TITLE TEXT [WIDTH] [HEIGHT]
```

As dimensões padrão são:

```text
360 x 300
```

No modo gráfico, a função usa o Zenity. Quando um terminal interativo está disponível como alternativa, ela solicita uma resposta `y/N`.

Exemplo:

```bash
if question "Example" "Install development tools?" 400 250; then
    pkg_install example-devel
fi
```

---

## Autenticação de Privilégios

### `askpass`

Para garantir que as credenciais do sudo estejam disponíveis:

```bash
askpass
```

ou explicitamente:

```bash
askpass sudo
```

Scripts legados podem usar:

```bash
sudo_rq
```

O LinuxToys primeiro verifica se a autorização existente do sudo ainda é válida. Se a autenticação for necessária, ele utiliza o método de senha apropriado para GUI ou CLI.

Em muitos casos, os desenvolvedores não precisam chamar isso diretamente porque os auxiliares da biblioteca invocam `sudo` por conta própria. É útil antes de uma sequência de comandos privilegiados quando a autenticação deve ocorrer de forma previsível antes do início da operação.

Não implemente caixas de diálogo de senha personalizadas nem encaminhe senhas capturadas arbitrariamente para `sudo`.

---

## Detecção do Sistema Operacional em Tempo de Execução

`sysinfo.bash` está sempre disponível.

Os principais auxiliares de distribuição são:

```bash
is_arch
is_cachy
is_fedora
is_ostree
is_debian
is_ubuntu
is_suse
is_solus
is_zorin
is_rhel
is_deepin
is_manjaro
is_systemd
```

Eles foram projetados para condicionais Bash comuns:

```bash
if is_fedora; then
    ...
elif is_arch; then
    ...
elif is_debian || is_ubuntu; then
    ...
fi
```

Prefira esses auxiliares a analisar `/etc/os-release` repetidamente.

### Exemplo

```bash
if is_debian || is_ubuntu; then
    pkg_install foo
elif is_fedora || is_rhel; then
    pkg_install foo
elif is_arch || is_cachy; then
    pkg_install foo
fi
```

Neste exemplo específico, as ramificações são desnecessárias se o pacote tiver o mesmo nome em todos os sistemas:

```bash
pkg_install foo
```

Use verificações de distribuição somente quando o procedimento realmente for diferente.

---

## Detecção de Hardware em Tempo de Execução

A biblioteca de informações do sistema também disponibiliza:

```bash
is_nvidia
is_intel
is_icr_capable
is_amd
amd_dgpu
rocm_apu
is_rocm_capable
has_rebar
is_hybridgpu
```

Por exemplo:

```bash
if is_nvidia; then
    pkg_install nvidia-settings
fi
```

or:

```bash
if is_rocm_capable; then
    ...
fi
```

A compatibilidade com Intel Compute Runtime é disponibilizada por:

```bash
is_icr_capable
```

enquanto a elegibilidade de hardware para ROCm é disponibilizada por:

```bash
is_rocm_capable
```

A biblioteca também disponibiliza `has_rebar` para verificar se uma GPU possui uma alocação BAR maior que a janela tradicional de 256 MiB, e `is_hybridgpu` para sistemas híbridos NVIDIA + Intel/AMD.

Use cabeçalhos quando hardware incompatível deve impedir completamente que o script apareça. Use detecção em tempo de execução quando o script oferecer suporte a várias configurações de hardware, mas precisar de comandos diferentes para cada uma.

---

## Gerenciamento de Pacotes

Uma das abstrações mais importantes do LinuxToys é a biblioteca de gerenciamento de pacotes.

Evite escrever manualmente grandes blocos como:

```bash
if is_debian; then
    sudo apt install ...
elif is_fedora; then
    sudo dnf install ...
elif is_arch; then
    sudo pacman -S ...
fi
```

quando o nome do pacote for idêntico.

Use:

```bash
pkg_install package-name
```

em vez disso.

---

### `pkg_install`

```bash
pkg_install curl git
```

Instala um ou mais pacotes nativos.

Antes de instalar, o LinuxToys verifica quais dos pacotes solicitados já estão instalados e os ignora.

A função atualmente abstrai a instalação de pacotes no Debian/Ubuntu, Arch/CachyOS/Manjaro, Fedora/RHEL, sistemas rpm-ostree, openSUSE e Solus. Instalações bem-sucedidas são adicionadas ao mapa de transações para que o LinuxToys possa desfazê-las posteriormente.

### Opções

`pkg_install` reconhece opções especiais do LinuxToys antes dos argumentos de pacote.

#### `--ignore-appends`

```bash
pkg_install --ignore-appends foo
```

Instala sem registrar a transação normal do pacote.

Use com moderação. Isso deliberadamente exclui essa operação do rastreamento normal de reversão.

#### `--ostreecheck`

```bash
pkg_install --ostreecheck foo
```

Em sistemas rpm-ostree, a instalação é realizada e o LinuxToys então trata o estado de implantação pendente.

#### `--allowerasing`

```bash
pkg_install --allowerasing foo
```

Ativa o caminho `--allowerasing` do DNF quando aplicável.

---

## Verificando se Pacotes Estão Instalados

### `pkg_exists`

```bash
pkg_exists foo bar
```

Preenche dois arrays:

```bash
pkg_found
pkg_notfound
```

Exemplo:

```bash
pkg_exists foo

if [[ ${#pkg_found[@]} -gt 0 ]]; then
    echo "foo is installed"
fi
```

Em código de instalação comum, chamar `pkg_exists` por conta própria geralmente é desnecessário, pois `pkg_install` já realiza essa verificação.

---

## Removendo Pacotes Nativos

### `pkg_remove`

```bash
pkg_remove foo bar
```

Remove pacotes instalados usando o gerenciador de pacotes nativo.

Também existe um alias de compatibilidade:

```bash
pkg_rm foo
```

A maioria dos scripts de instalação não deve remover manualmente seus próprios pacotes durante a desinstalação normal pelo LinuxToys; instalações de pacotes registradas no mapa de transações podem ser tratadas pelo sistema de reversão.

`pkg_remove` é útil quando a remoção de pacotes faz parte do próprio procedimento de instalação.

---

## Instalação de Flatpak

### `pkg_flat`

```bash
pkg_flat org.example.Application
```

Garante que o suporte a Flatpak/Flathub exista, instala ou atualiza o aplicativo especificado, verifica o resultado e registra Flatpaks recém-instalados no mapa de transações.

Por padrão, o LinuxToys prefere a instalação Flatpak do usuário quando apropriado.

Para solicitar a instalação no sistema:

```bash
pkg_flat --skip-user org.example.Application
```

Como aplicativos Flatpak não podem ser instalados de dentro dos contêineres aos quais o LinuxToys se destina, o uso de `pkg_flat` marca automaticamente o script como incompatível com contêineres.

---

## Instalando um Pacote Baixado

### `pkg_fromfile`

```bash
pkg_fromfile ./example.deb
```

or:

```bash
pkg_fromfile ./example.rpm
```

Instala um pacote local usando o gerenciador de pacotes apropriado para a distribuição atual.

O auxiliar também trata arquivos `.flatpak`.

Isso é útil quando o script baixa ou compila um pacote antes de instalá-lo.

---

## Instalando Diretamente de uma URL

### `pkg_fromurl`

```bash
pkg_fromurl \
    "https://example.org/releases/example-amd64.deb"
```

Múltiplas URLs são aceitas:

```bash
pkg_fromurl \
    "https://example.org/foo.deb" \
    "https://example.org/foo-data.deb"
```

A função cria um diretório temporário para os downloads, baixa cada URL utilizando `curl` e, em seguida, passa o arquivo resultante para `pkg_appimage` ou `pkg_fromfile`, dependendo de seu tipo.

Isso geralmente é preferível a combinar manualmente:

```bash
wget ...
sudo dpkg -i ...
```

pois o LinuxToys mantém seu comportamento normal de gerenciamento de pacotes e transações.

#### Instalando Aplicativos em Tarballs

Para aplicativos não empacotados distribuídos como arquivos `.tar.gz` ou `.tar.xz` binários pré-compilados, passe o argumento `--tar`:

```bash
pkg_fromurl --tar \
    "https://example.org/releases/example-linux-x86_64.tar.xz"
```

Nesse modo, o arquivo baixado é passado para o manipulador de tarballs do LinuxToys em vez do instalador normal de pacotes.

O aplicativo é extraído em:

```text
~/.local/linuxtoys/apps
```

Se o arquivo já contiver todo o seu conteúdo dentro de um único diretório de nível superior, esse diretório será utilizado diretamente. Caso contrário, o LinuxToys criará um diretório para o aplicativo com base no nome do tarball.

Executar novamente a instalação substitui o diretório anterior do aplicativo em vez de mesclar arquivos com ele, permitindo que o mesmo comando seja utilizado para atualizações sem deixar arquivos obsoletos para trás.

`--tarball` também é aceito como um alias para `--tar`.

---

## Instalando o Release Mais Recente do GitHub/Codeberg

### `pkg_fromrelease`

Para projetos que publicam pacotes instaláveis em releases do GitHub ou Codeberg:

```bash
pkg_fromrelease \
    "https://github.com/example/example"
```

Um glob opcional para o asset pode ser utilizado para restringir o resultado:

```bash
pkg_fromrelease \
    "https://github.com/example/example" \
    "*desktop*"
```

O helper consulta o release estável mais recente, detecta a arquitetura da máquina e o formato de pacote nativo, filtra assets inadequados e seleciona um pacote instalável do release antes de passá-lo para `pkg_fromurl`.

Os formatos de release reconhecidos incluem AppImage, Flatpak e o formato de pacote nativo apropriado.

Em x86-64, assets de release para x86 de 32 bits podem ser utilizados como fallback apenas quando não houver um candidato adequado de 64 bits ou independente de arquitetura.

#### Instalando Tarballs de Releases

Para projetos que distribuem um aplicativo pré-compilado como um asset `.tar.gz` ou `.tar.xz` de um release, use `--tar`:

```bash
pkg_fromrelease --tar \
    "https://github.com/example/example"
```

Um glob para o asset ainda pode ser fornecido quando necessário:

```bash
pkg_fromrelease --tar \
    "https://github.com/example/example" \
    "*linux*"
```

Com `--tar`, a seleção do release é restrita a assets `.tar.gz` e `.tar.xz` compatíveis. O arquivo selecionado é então passado para `pkg_fromurl` no modo tarball e instalado em:

```text
~/.local/linuxtoys/apps
```

Esse modo é destinado a **tarballs binários pré-compilados de aplicativos**, e não a arquivos de código-fonte. Os arquivos de código-fonte gerados automaticamente pelo GitHub não são considerados, pois não são assets enviados ao release, e assets identificados como arquivos de código-fonte são filtrados da seleção.

A filtragem por arquitetura continua sendo aplicada no modo tarball, portanto arquivos binários específicos para cada arquitetura são selecionados de acordo com a máquina atual sempre que possível.

`--tarball` também é aceito como um alias para `--tar`.

---

## Integração de AppImage

### `pkg_appimage`

```bash
pkg_appimage ./Example.AppImage
```

Integra um ou mais AppImages ao sistema do usuário e os registra para reversão.

A função também trata atualizações: se o registro atual do LinuxToys indicar que esse script instalou anteriormente um AppImage, o LinuxToys identifica o nome do arquivo integrado anterior e o remove antes de integrar seu substituto.

Em sistemas com systemd, o LinuxToys atualmente realiza a integração de AppImages por meio do Gear Lever. Em outros sistemas, utiliza seu método de integração manual.

Instaladores AppImage são automaticamente excluídos de ambientes em contêineres.

---

## Pacotes npm e Bun

### `pkg_npm`

```bash
pkg_npm package-name
```

Instala o npm se necessário, configura o caminho global do usuário quando necessário, ignora pacotes já instalados globalmente e registra os pacotes npm instalados.

---

### `pkg_bun`

```bash
pkg_bun package-name
```

Garante que o Bun esteja instalado ou o atualiza, configura o PATH relevante do usuário, instala pacotes globais ausentes e os registra.

---

## Operações do Sistema de Arquivos e Reversão

Para arquivos que o LinuxToys possa precisar restaurar posteriormente, use os auxiliares de preparação do sistema de arquivos em vez de editá-los diretamente sem registro.

Essas funções são fundamentais para o rollback automático.

---

### `prep_create`

Antes de criar um novo arquivo:

```bash
prep_create /etc/example/example.conf

sudo tee /etc/example/example.conf >/dev/null <<'EOF'
option=true
EOF
```

`prep_create`:

1. cria diretórios pais ausentes, se necessário;
2. cria o arquivo;
3. registra:

```text
created /etc/example/example.conf
```

no mapa de transações.

Se o arquivo inesperadamente já existir, o LinuxToys passa a usar `prep_edit` em vez de destruir o original.

---

## Preparando um Arquivo Existente para Modificação

### `prep_edit`

```bash
prep_edit /etc/example/example.conf

sudo sed -i 's/foo/bar/' /etc/example/example.conf
```

Antes da edição, o LinuxToys cria uma cópia `.bak` e registra que o arquivo foi editado.

Se o arquivo esperado não existir, ele registra um aviso e passa a tratar o arquivo como recém-criado.

Isso é preferível a:

```bash
sudo cp /etc/example/example.conf /etc/example/example.conf.bak
```

porque o sistema de transações compreende `prep_edit`.

---

## Preparando um Arquivo ou Diretório para Remoção

### `prep_rm`

```bash
prep_rm /etc/example/obsolete.conf
```

Em vez de excluir imediatamente o alvo, o LinuxToys o move para um caminho `.bak` e registra a remoção.

Isso torna a operação reversível.

---

## Criando Diretórios

### `prep_dir`

```bash
prep_dir /etc/example
```

Cria o diretório somente se ele ainda não existir e registra os diretórios recém-criados.

---

### `prep_dir_edit`

Antes de modificar um diretório existente como uma unidade:

```bash
prep_dir_edit /etc/example
```

O LinuxToys cria um backup dele como:

```text
/etc/example.bak
```

e o marca como editado.

---

## Diretórios Temporários

### `prep_tmp`

```bash
prep_tmp
```

Seleciona e entra no diretório temporário de trabalho do LinuxToys, recorrendo à localização temporária configurada, `/tmp/linuxtoys` e, por fim:

```text
$HOME/.cache/linuxtoys/tmp
```

---

### `prep_tmp_noram`

```bash
prep_tmp_noram
```

Usa explicitamente:

```text
$HOME/.cache/linuxtoys/tmp
```

Isso é útil para downloads/compilações maiores, nos quais armazenamento temporário baseado em RAM seria inadequado.

---

## Auxiliares de Cópia e Movimentação com Detecção de Privilégios

### `copy_`

```bash
copy_ source destination
```

or:

```bash
copy_ -r source-directory destination
```

O LinuxToys primeiro tenta realizar a cópia como o usuário atual e tenta novamente por meio de `sudo` se necessário. Uma origem ausente ou uma falha na cópia é tratada como fatal.

---

### `move_`

```bash
move_ source destination
```

Funciona de maneira semelhante para movimentações, com fallback automático para sudo.

---

## Auxiliares do systemd

Quando um script precisar gerenciar serviços do systemd, use as funções `sysd_*`.

### Serviços do sistema

```bash
sysd_enable example.service
sysd_start example.service
```

Operações disponíveis:

```text
sysd_enable
sysd_disable
sysd_start
sysd_stop
```

LinuxToys performs a daemon reload where needed and records each operation no mapa de transações.

Múltiplos serviços podem ser fornecidos:

```bash
sysd_enable example.service example.timer
sysd_start example.service example.timer
```

---

### Serviços systemd do Usuário

Para serviços do usuário:

```bash
sysd_enable_usr example.service
sysd_start_usr example.service
```

As funções disponíveis são:

```text
sysd_enable_usr
sysd_disable_usr
sysd_start_usr
sysd_stop_usr
```

Elas invocam:

```text
systemctl --user
```

em vez do gerenciador de serviços do sistema.

---

## Gerenciamento do Carregador de Inicialização e initramfs

### `bootloader_upd`

```bash
bootloader_upd
```

Atualiza o carregador de inicialização instalado usando o método apropriado para a distribuição atual.

O LinuxToys atualmente trata variantes do GRUB e, quando aplicável, Limine, systemd-boot, `sdboot-manage` e o `clr-boot-manager` do Solus. Sistemas rpm-ostree ignoram o caminho normal de regeneração do carregador de inicialização.

Use isso em vez de espalhar comandos específicos de carregador de inicialização para cada distribuição pelo script.

---

### `initramfs_upd`

```bash
initramfs_upd
```

Regenera imagens initramfs usando a implementação apropriada:

```text
Debian/Ubuntu      update-initramfs
Arch/CachyOS       dracut or mkinitcpio
Fedora/RHEL/SUSE   dracut
```

The operation is recorded no mapa de transações.

---

## Argumentos do Kernel

### `kargs_upd`

Para sistemas rpm-ostree:

```bash
kargs_upd "example.option=1"
```

Múltiplos argumentos podem ser passados:

```bash
kargs_upd \
    "example.option=1" \
    "another.option=2"
```

Cada um é adicionado com `rpm-ostree kargs --append` e registrado.

---

### `grubbyargs_upd`

```bash
grubbyargs_upd "example.option=1"
```

Garante que `grubby` exista e adiciona cada argumento a todos os kernels.

---

## Secure Boot

### `secureboot_check`

```bash
secureboot_check
```

Verifica se o Secure Boot está ativo e realiza a preparação apropriada do LinuxToys para as distribuições suportadas.

A implementação atualmente inclui caminhos para Fedora/RHEL/rpm-ostree, Ubuntu e Debian. Sistemas da família Fedora podem delegar ao script de assinatura de módulos do LinuxToys, enquanto os caminhos Ubuntu/Debian trabalham com inscrição MOK.

Para procedimentos no Debian destinados a usar a localização MOK no estilo do Ubuntu:

```bash
secureboot_check --ubuntumok
```

Use esse auxiliar antes de instalar módulos de kernel não assinados, em vez de reimplementar o fluxo de Secure Boot do LinuxToys em cada instalador de driver.

---

## Auxiliares Diversos de Transação

Algumas operações não se encaixam no modelo de pacotes/sistema de arquivos, mas ainda precisam ser registradas.

### `shell_change`

```bash
shell_change /bin/zsh
```

Altera o shell do usuário atual e registra a operação.

---

### `distrobox_created`

```bash
distrobox_created example-container
```

Records a Distrobox creation no mapa de transações.

A criação efetiva do contêiner é realizada separadamente.

---

### `swapfile_created`

```bash
swapfile_created /swapfile
```

Registra um arquivo swap criado para reversão.

---

### `rclone_mount`

```bash
rclone_mount myremote /mnt/example
```

Monta um remoto do rclone como daemon e registra a montagem resultante.

---

## Substituições do Flatpak

### `flatpak_override`

Exemplo:

```bash
flatpak_override user fs "$HOME/Games" org.example.Application
```

Forma geral:

```text
flatpak_override SCOPE TYPE SETTING APPLICATION
```

Escopo:

```text
user
system
```

Os aliases de tipo aceitos incluem:

```text
fs      -> filesystem
name    -> talk-name
dbus    -> talk-dbus
```

e os tipos diretos:

```text
share
env
runtime
device
socket
filesystem
talk-name
talk-dbus
```

A função verifica se o Flatpak alvo existe e registra a substituição para reversão.

---

## Chamando Outro Script do LinuxToys

### `call_script`

Scripts do LinuxToys podem reutilizar outros scripts do LinuxToys:

```bash
call_script rpmfusion
```

O sufixo `.sh` é opcional:

```bash
call_script rpmfusion.sh
```

O LinuxToys procura o script indicado em seu cache sincronizado de scripts, verifica se ele é compatível com a máquina atual e somente então o executa. Um filho incompatível é ignorado com status de retorno `2`.

Argumentos podem ser encaminhados:

```bash
call_script child-script --some-option value
```

### Comportamento das transações de scripts filhos

Scripts chamados recebem seu próprio mapa de transações.

Se o filho for bem-sucedido:

1. sua transação é confirmada independentemente;
2. o filho é salvo no registro;
3. a transação do pai registra que o filho foi chamado.

Se o filho falhar, suas operações de transação ainda não confirmadas são mescladas à transação do pai para que o rollback normal de nível superior possa desfazer essas alterações.

Isso permite decompor funcionalidades complexas em recursos reutilizáveis do LinuxToys sem perder a segurança do rollback.

Prefira:

```bash
call_script paru
```

em vez de duplicar toda a lógica de instalação de outro script do LinuxToys.

---

## Biblioteca Auxiliar Opcional

Alguns utilitários menos fundamentais são mantidos na biblioteca auxiliar adicional.

Carregue-a com:

```bash
summon_helpers
```

O LinuxToys então carrega sua biblioteca auxiliar.

Os utilitários disponibilizados atualmente incluem:

```text
fetch_from_mirror
multilib_chk
clinfo_chk
enable_debian_nonfree
enable_debian_backports
```

além de wrappers legados de compatibilidade, incluindo:

```text
chaotic_aur_lib
rpmfusion_chk
pip_lib
flatpak_in_lib
```

Os wrappers legados delegam para scripts normais do LinuxToys por meio de `call_script`.

### Exemplo

```bash
summon_helpers

if is_debian; then
    enable_debian_nonfree
fi
```

---

## Biblioteca Auxiliar de Otimizadores

Auxiliares especializados de otimização podem ser carregados com:

```bash
summon_optimizers
```

A biblioteca de otimizadores atual fornece principalmente chamadas reutilizáveis para scripts de otimização existentes do LinuxToys, além de funcionalidades especializadas como a configuração de CDI da NVIDIA.

Exemplos de funções disponibilizadas atualmente incluem:

```text
nvidia_ctkpatch
cachyos_sysd_lib
sboost_lib
preempt_lib
dsplitm_lib
psave_lib
earlyoom_lib
zswap_lib
wayland_proton_lib
intel_xe_lib
free_mem_fix
dnsmasq_lib
fix_intel_gtk
pp_ondemand
```

A maioria dos wrappers legados de otimização agora simplesmente chama o script correspondente do LinuxToys.

Para código novo, prefira `call_script` diretamente quando nenhuma abstração adicional for necessária.

---

## O Mapa de Transações

O modelo de reversão do LinuxToys gira em torno do mapa de transações.

Um mapa de transações é inicializado sob demanda quando algo precisa registrar uma operação pela primeira vez. O LinuxToys cria o arquivo com modo `600` e adiciona operações por meio de `_append_transmap`.

Normalmente, os desenvolvedores **não** devem escrever manualmente no mapa de transações.

Em vez disso, use funções com suporte a transações, como:

```text
pkg_install
pkg_flat
pkg_fromfile
pkg_appimage

prep_create
prep_edit
prep_rm
prep_dir
prep_dir_edit

sysd_enable
sysd_start

bootloader_upd
initramfs_upd

shell_change
flatpak_override
```

Essas funções realizam a operação e também informam ao LinuxToys como o sistema foi alterado.

Esta é uma das regras mais importantes ao desenvolver um script completo do LinuxToys:

> Se o LinuxToys já fornece um auxiliar com suporte a transações para uma operação, prefira-o ao comando shell bruto equivalente.

For example, prefer:

```bash
prep_edit /etc/example.conf
sudo sed -i 's/foo/bar/' /etc/example.conf
```

over:

```bash
sudo cp /etc/example.conf /etc/example.conf.bak
sudo sed -i 's/foo/bar/' /etc/example.conf
```

and prefer:

```bash
pkg_install foo
```

over:

```bash
sudo dnf install -y foo
```

Isso torna a recuperação de erros e a desinstalação posterior consideravelmente mais confiáveis.

---

## Tratamento de Erros e Scripts Compatíveis com Rollback

Comandos cuja falha signifique que a instalação não pode continuar com segurança devem encerrar por meio de `die`/`fatal`.

Recomendado:

```bash
some_command || die "Failed to configure Example."
```

Não recomendado:

```bash
some_command
```

quando o restante do script pressupõe que o comando foi bem-sucedido.

Da mesma forma, não oculte falhas desnecessariamente:

```bash
some_command || true
```

a menos que a falha seja realmente aceitável.

Um bom script do LinuxToys deve deixar o registro de transações em um estado que represente com precisão tudo o que foi alterado com sucesso antes da ocorrência de uma falha.

---

## Exemplo: Cross-Distribution Application Installer

```bash
# name: Example Server
# version: 1.0
# description: Installs and configures Example Server.
# icon: example-server.svg
# repo: https://github.com/example/example-server
# compat: debian, ubuntu, fedora, rhel, arch, cachy, suse
# systemd: yes
# nocontainer
# reboot: no
# revert: yes

# Install common dependencies.
pkg_install curl

# Package names differ between families.
if is_debian || is_ubuntu; then
    pkg_install example-server example-utils

elif is_fedora || is_rhel; then
    pkg_install example-server example-tools

elif is_arch || is_cachy; then
    pkg_install example-server example-tools

elif is_suse; then
    pkg_install example-server
fi

# Preserve an existing configuration or register a new one.
config="/etc/example/example.conf"

if [[ -f "$config" ]]; then
    prep_edit "$config"
else
    prep_create "$config"
fi

sudo tee "$config" >/dev/null <<'EOF'
enabled=true
port=1234
EOF

# Enable and start the service using transaction-aware helpers.
sysd_enable example.service
sysd_start example.service

info "$finishmsg"
```

Esse script faz com que o LinuxToys carregue automaticamente os módulos de pacotes, sistema de arquivos e systemd porque essas funções são referenciadas diretamente.

---

## Exemplo: Latest Upstream Release

Para um projeto cujo upstream publica pacotes prontos:

```bash
# name: Example Desktop
# version: 1.0
# description: Installs the latest stable version of Example Desktop.
# icon: example.svg
# repo: https://github.com/example/example
# compat: debian, ubuntu, fedora, rhel, arch, cachy
# reboot: no
# revert: yes

pkg_fromrelease "https://github.com/example/example"

info "$finishmsg"
```

O LinuxToys seleciona automaticamente o formato da distribuição e a arquitetura.

Se a release possuir vários assets igualmente válidos:

```bash
pkg_fromrelease \
    "https://github.com/example/example" \
    "*desktop*"
```

pode restringir a seleção.

---

## Exemplo: Driver Installer

```bash
# name: Example Kernel Driver
# version: 1.0
# description: Installs the Example hardware driver.
# icon: driver.svg
# compat: debian, ubuntu, fedora, rhel, arch, cachy
# deviceids: 1234:abcd
# systemd: yes
# nocontainer
# reboot: yes
# revert: yes

secureboot_check

if is_debian || is_ubuntu; then
    pkg_install dkms linux-headers-amd64
elif is_fedora || is_rhel; then
    pkg_install dkms kernel-devel
elif is_arch || is_cachy; then
    pkg_install dkms linux-headers
fi

prep_tmp_noram

git clone https://github.com/example/example-driver.git \
    || die "Failed to download Example driver."

cd example-driver \
    || die "Failed to enter driver source directory."

sudo ./install.sh \
    || die "Failed to install Example driver."

initramfs_upd

info "$rebootmsg"
```

Os nomes exatos dos pacotes em um script real de driver devem, naturalmente, refletir as distribuições efetivamente suportadas pelo driver.

---

## Escolhendo entre Cabeçalhos e Verificações em Tempo de Execução

Use um **cabeçalho** quando máquinas incompatíveis nunca devem receber a opção do recurso:

```bash
# gpu: nvidia
```

Use uma **função em tempo de execução** quando o script oferecer suporte a múltiplos sistemas, mas precisar de comportamentos diferentes:

```bash
if is_nvidia; then
    ...
elif is_amd; then
    ...
fi
```

Uma regra útil é:

```text
Header        -> Can this machine use the feature at all?
Runtime check -> How should the feature be installed on this machine?
```

Por exemplo:

```bash
# compat: debian, ubuntu, fedora, arch
```

combined with:

```bash
if is_debian || is_ubuntu; then
    ...
elif is_fedora; then
    ...
elif is_arch; then
    ...
fi
```

é apropriado quando o software oferece suporte às quatro famílias, mas exige procedimentos diferentes.

---

## Práticas Recomendadas

Ao escrever um script do LinuxToys:

1. **Mantenha todos os cabeçalhos no início do arquivo.** A análise do cabeçalho é encerrada na primeira linha que não for um comentário.

2. **Use cabeçalhos de compatibilidade em vez de abortar manualmente em sistemas não suportados.** O usuário não deve receber a opção de um recurso que o LinuxToys já sabe ser incompatível.

3. **Use `pkg_install` em vez de chamar gerenciadores de pacotes diretamente** sempre que os nomes dos pacotes permitirem.

4. **Use `prep_create`, `prep_edit`, `prep_rm`, `prep_dir` e `prep_dir_edit` para alterações persistentes no sistema de arquivos.** Essas funções tornam o rollback possível.

5. **Use os auxiliares `sysd_*` para alterações em serviços.**

6. **Use `bootloader_upd` e `initramfs_upd` em vez de duplicar lógica de inicialização específica de cada distribuição.**

7. **Use `secureboot_check` para instaladores de drivers/módulos que precisam tratar o Secure Boot.**

8. **Use `call_script` quando o LinuxToys já implementar um pré-requisito como outro script.** Não copie o instalador dele para o seu.

9. **Use `die` para falhas fatais.** Um script não deve continuar depois que uma operação pré-requisito falhar.

10. **Mantenha `revert` ativado sempre que possível.** As funções do LinuxToys com suporte a transações são projetadas em torno de instalações reversíveis.

11. **Não adicione restrições de compatibilidade desnecessárias.** A ausência de um cabeçalho de GPU, CPU, desktop, Wayland ou semelhante significa que aquela dimensão não possui restrições.

12. **Não carregue manualmente as bibliotecas principais modulares.** Scripts comuns são inspecionados pelo `library_loader.py` e recebem automaticamente os módulos que utilizam.

---

## Referência Rápida das Funções Principais

| Área                       | Funções                                                                                                                                                        |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Mensagens                  | `info`, `warn`, `error`, `die`, `question`                                                                                                                       |
| Autenticação               | `askpass`, `sudo_rq`                                                                                                                                             |
| Detecção de SO             | `is_arch`, `is_cachy`, `is_fedora`, `is_ostree`, `is_debian`, `is_ubuntu`, `is_suse`, `is_solus`, `is_zorin`, `is_rhel`, `is_deepin`, `is_manjaro`, `is_systemd` |
| Hardware                   | `is_nvidia`, `is_intel`, `is_icr_capable`, `is_amd`, `amd_dgpu`, `rocm_apu`, `is_rocm_capable`, `has_rebar`, `is_hybridgpu`                                      |
| Pacotes nativos            | `pkg_exists`, `pkg_install`, `pkg_remove`, `pkg_rm`                                                                                                              |
| Pacotes portáteis/externos | `pkg_flat`, `pkg_fromfile`, `pkg_fromurl`, `pkg_fromrelease`, `pkg_appimage`, `pkg_appimage_rm`                                                                  |
| Gerenciadores de pacotes de linguagens | `pkg_npm`, `pkg_bun`                                                                                                                                             |
| Sistema de arquivos        | `prep_create`, `prep_edit`, `prep_rm`, `prep_dir`, `prep_dir_edit`, `prep_tmp`, `prep_tmp_noram`, `copy_`, `move_`                                               |
| systemd                    | `sysd_enable`, `sysd_disable`, `sysd_start`, `sysd_stop`, `sysd_enable_usr`, `sysd_disable_usr`, `sysd_start_usr`, `sysd_stop_usr`                               |
| Inicialização              | `bootloader_upd`, `initramfs_upd`, `kargs_upd`, `grubbyargs_upd`, `secureboot_check`                                                                             |
| Diversos                   | `shell_change`, `distrobox_created`, `rclone_mount`, `swapfile_created`, `flatpak_override`                                                                      |
| Composição                 | `call_script`                                                                                                                                                    |
| Bibliotecas opcionais      | `summon_helpers`, `summon_optimizers`                                                                                                                            |

---

## Referência Rápida Completa dos Cabeçalhos

| Cabeçalho        | Finalidade                  | Exemplo                            |
| ---------------- | -------------------------- | ---------------------------------- |
| `name`           | Nome do recurso exibido ao usuário   | `# name: Example`                  |
| `version`        | Versão dos metadados do script    | `# version: 1.0`                   |
| `description`    | Descrição exibida ao usuário    | `# description: Installs Example.` |
| `icon`           | Ícone do recurso               | `# icon: example.svg`              |
| `repo`           | Repositório upstream        | `# repo: https://github.com/...`   |
| `compat`         | Compatibilidade de SO           | `# compat: fedora, arch, !manjaro` |
| `gpu`            | Compatibilidade de GPU          | `# gpu: amd, intel`                |
| `cpu`            | Compatibilidade de CPU          | `# cpu: amd`                       |
| `desktop`        | Compatibilidade de desktop      | `# desktop: gnome, plasma`         |
| `systemd`        | Requisito do sistema de init    | `# systemd: yes`                   |
| `wayland`        | Requisito de sessão        | `# wayland: yes`                   |
| `hybridgpu`      | Filtragem de GPU híbrida       | `# hybridgpu: only`                |
| `deviceids`      | Filtragem de hardware PCI/USB | `# deviceids: 1234:abcd`           |
| `nocontainer`    | Filtragem de contêiner        | `# nocontainer: fedora`            |
| `localize`       | Filtragem por localidade           | `# localize: pt, es`               |
| `reboot`         | Requisito de reinicialização         | `# reboot: yes`                    |
| `revert`         | Política de reversão           | `# revert: yes`                    |
| `negates`        | Ocultar scripts substituídos    | `# negates: old-script`            |
| `optimized-only` | Filtragem por conjunto de otimização | `# optimized-only:`                |

---

## Modelo Mínimo Final

Para um novo instalador completo do LinuxToys, este é um bom ponto de partida:

```bash
# name: Application Name
# version: 1.0
# description: Installs and configures Application Name.
# icon: application.svg
# repo: https://github.com/vendor/application
# compat: debian, ubuntu, fedora, arch
# reboot: no
# revert: yes

# Install dependencies/packages.
pkg_install package-name

# Persistent files should be registered before modification.
config="/etc/application/application.conf"

if [[ -f "$config" ]]; then
    prep_edit "$config"
else
    prep_create "$config"
fi

# Perform configuration.
sudo tee "$config" >/dev/null <<'EOF'
example=true
EOF

info "$finishmsg"
```

A partir daí, adicione apenas os cabeçalhos de compatibilidade e as funções de biblioteca realmente exigidos pelo recurso.

O princípio central de design é simples:

> **Deixe o LinuxToys cuidar da portabilidade, compatibilidade e rastreamento de transações sempre que um auxiliar existente puder fazê-lo. Mantenha o script focado no procedimento específico do software.**
