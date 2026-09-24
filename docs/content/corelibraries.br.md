# Bibliotecas Principais do LinuxToys

Os scripts de procedimento do LinuxToys são intencionalmente pequenos. Operações comuns — gerenciamento de pacotes, alterações no sistema de arquivos, serviços, privilégios administrativos, detecção do sistema, diálogos e rastreamento de transações — são fornecidas pelas bibliotecas shell do LinuxToys. Ele também utiliza suas bibliotecas para procedimentos padrão como a instalação de aplicativos *AppStream* - é o que permite que funcione mesmo no *Arch Linux* e derivados, que não funcionam bem com *PackageKit*.

**Você não precisa importar essas bibliotecas manualmente.**

Quando o LinuxToys inicia um procedimento, o carregador analisa o script antes da execução, detecta as funções de biblioteca utilizadas, acompanha dependências entre bibliotecas, define os indicadores necessários e carrega o núcleo do LinuxToys automaticamente. `sysinfo.bash` está sempre disponível; os outros módulos são carregados apenas quando necessários.

```bash
pkg_install curl git
prep_create "$HOME/.config/example/config"
info "$finishmsg"
```

Não é necessário adicionar `source`, blocos de importação ou inicialização de bibliotecas ao script.

## Como as bibliotecas são carregadas

Antes da execução, o LinuxToys faz uma detecção estática conservadora dos nomes de funções referenciados pelo script. Ele também inspeciona hooks estáticos chamados com `run_list_hook` e acompanha dependências transitivas entre as bibliotecas.

| Biblioteca | Indicador interno |
| --- | --- |
| `fsops.bash` | `FS_OPS` |
| `packages.bash` | `PACKAGE_OPS` |
| `boot.bash` | `BOOT_OPS` |
| `misc.bash` | `MISC_OPS` |
| `sysd.bash` | `SYSD_OPS` |
| `helpers.bash` | `HELPERS_OPS` |
| `optimizers.bash` | `OPTIMIZER_OPS` |

Depois, o carregador importa `linuxtoys.bash`, que sempre carrega `sysinfo.bash` e carrega condicionalmente os módulos selecionados. A biblioteca systemd só é carregada quando o sistema realmente usa systemd.

A detecção prefere carregar uma biblioteca desnecessária a deixar de carregar uma necessária. Nomes de funções construídos dinamicamente não podem ser inferidos; nesse caso incomum, o indicador `*_OPS` apropriado ainda pode ser definido explicitamente antes do carregamento do núcleo.

As chamadas antigas `summon_helpers` e `summon_optimizers` continuam disponíveis por compatibilidade, mas scripts novos devem simplesmente chamar as funções necessárias.

## Biblioteca principal

`linuxtoys.bash` fornece recursos compartilhados por praticamente todos os procedimentos: mensagens localizadas, diálogos, elevação de privilégios, execução de scripts aninhados, integração com o runner e rastreamento de transações.

### Privilégios administrativos

Use:

```bash
sudo_ comando argumentos...
```

em vez de chamar `sudo` diretamente.

`sudo_` valida a autenticação imediatamente antes da elevação e usa o fluxo gráfico ou de terminal do LinuxToys conforme apropriado. Normalmente não é necessário chamar `askpass` manualmente.

### Informações, avisos e erros

```bash
info "Operação concluída."
warn "Esta configuração é incomum."
error "A operação opcional falhou."
die "A instalação não pode continuar."
```

`info` e `warn` usam um diálogo gráfico quando apropriado e caem para o terminal quando necessário. `error` informa um erro não fatal e retorna falha. `die` informa um erro fatal e encerra o procedimento.

Os aliases antigos `zeninf`, `zenwrn`, `nonfatal` e `fatal` continuam disponíveis.

### Perguntas

```bash
if question "$msg006" "Prosseguir com a instalação?"; then
    # aceito
else
    # cancelado
fi
```

Formato:

```text
question TÍTULO TEXTO [LARGURA] [ALTURA]
```

O tamanho padrão é 360×300. Sem diálogo gráfico, o LinuxToys usa automaticamente um prompt `[y/N]` no terminal.

### Seleção única

```bash
choice=$(radioselect "Opção A" "Opção B" "Opção C") || exit 1
```

### Seleção múltipla

```bash
mapfile -t choices < <(
    listselect "Recurso A" "Recurso B" "Recurso C"
)
```

`listselect` imprime uma seleção por linha.

### Chamar outro procedimento do LinuxToys

```bash
call_script flathub
```

Use `call_script` para reutilizar um procedimento existente. Scripts aninhados passam pelo mesmo carregamento automático de bibliotecas.

### Hooks de listas

```bash
run_list_hook path/to/hook.sh
```

Hooks referenciados estaticamente também participam da detecção automática de dependências.

## Variáveis de mensagens localizadas e reutilizáveis

O LinuxToys carrega automaticamente a biblioteca de idioma de acordo com o locale atual. Scripts podem reutilizar variáveis genéricas já traduzidas em vez de embutir mensagens em inglês.

| Variável | Significado em inglês | Uso típico |
| --- | --- | --- |
| `$msg006` | `Installer` | Título genérico de instalador/pergunta |
| `$finishmsg` | `Operations completed.` | Conclusão genérica com sucesso |
| `$msg022` | `Reboot to take effect.` | Aviso curto de reinicialização |
| `$rebootmsg` | `Installation complete. Reboot for changes to take effect.` | Instalação concluída que exige reinicialização |
| `$msg024` | `Repository enabled successfully.` | Repositório habilitado |
| `$cancelmsg` | `Cancel` | Texto genérico de cancelamento |
| `$incompatmsg` | `Your operating system is not compatible.` | Incompatibilidade genérica do sistema operacional |
| `$abortmsg` | `Operation cancelled by the user.` | Cancelamento pelo usuário |
| `$msg276` | `Understood.` | Confirmação genérica |
| `$msg277` | `Select versions to install` | Seleção de versões |
| `$notdomsg` | `Nothing to do.` | Operação sem alterações necessárias |
| `$rmmsg` | `You already have $LT_PROGRAM installed. Do you wish to remove it?` | Pergunta genérica de remoção após definir `LT_PROGRAM` |
| `$outofspace` | `Insufficient available storage space for operation.` | Espaço insuficiente |
| `$hwincompat` | `Your hardware is not compatible with this feature. Operation cancelled.` | Incompatibilidade de hardware |
| `$parumsg` | Mensagem explicando que `paru` é necessário e será instalado | Configuração do AUR |
| `$gearlevermsg` | Mensagem explicando que Gear Lever é necessário e será instalado | Integração de AppImages |

Exemplo:

```bash
if ! is_ubuntu; then
    die "$incompatmsg"
fi

if question "$msg006" "Este recurso altera a configuração de boot. Prosseguir?"; then
    # operação
else
    info "$abortmsg"
    exit 0
fi

info "$finishmsg"
```

Só reutilize uma variável quando o significado realmente corresponder. Mensagens específicas de um aplicativo ou contexto devem continuar tendo sua própria entrada de tradução.

## Informações e compatibilidade do sistema

`sysinfo.bash` está sempre carregada.

### Distribuições

```bash
is_arch
is_cachy
is_manjaro
is_fedora
is_ostree
is_debian
is_ubuntu
is_suse
is_solus
is_zorin
is_rhel
is_deepin
is_steamos
```

São predicados shell:

```bash
if is_fedora || is_ostree; then
    # tratamento da família Fedora
fi
```

### Sistema init

```bash
is_systemd
```

### GPU e hardware

```bash
is_nvidia
is_intel
is_amd
is_hybridgpu
is_icr_capable
is_rocm_capable
has_rebar
```

## Operações de sistema de arquivos

Esses helpers integram alterações ao mapa de transações do LinuxToys.

### Editar um arquivo

```bash
prep_edit /etc/example.conf
sudo_ sed -i 's/old/new/' /etc/example.conf
```

### Criar um arquivo

```bash
prep_create /etc/example.conf
```

Se o arquivo já existir inesperadamente, o LinuxToys o trata como edição em vez de sobrescrever dados sem rastreamento.

### Remover arquivo ou diretório

```bash
prep_rm /etc/example.conf
```

### Diretórios

```bash
prep_dir /etc/example
prep_dir_edit /etc/example
```

### Diretório temporário

```bash
prep_tmp
```

ou:

```bash
prep_tmp_noram
```

### Copiar e mover

```bash
copy_ source destination
copy_ -r directory destination
move_ source destination
```

Os helpers tentam a operação sem privilégios primeiro e elevam apenas quando necessário.

## Gerenciamento de pacotes

### Pacotes nativos

```bash
pkg_install curl git
```

O LinuxToys escolhe o gerenciador apropriado e ignora pacotes já instalados.

Opções úteis:

```bash
pkg_install --no-recommends package
pkg_install --allowerasing package
pkg_install --ostreecheck package
pkg_install --ignore-appends package
pkg_install --bypass package
```

`--bypass` desativa o bloqueio do runner para a transação, sendo destinado a operações que precisam permanecer interativas.

Remoção:

```bash
pkg_remove package
```

`pkg_rm` continua como alias.

### Flatpak

```bash
pkg_flat org.example.App
```

### Arquivos de pacote

```bash
pkg_fromfile ./package.deb
pkg_fromfile ./package.rpm
pkg_fromfile ./package.pkg.tar.zst
pkg_fromfile ./application.flatpak
```

Flatpak em escopo de sistema:

```bash
pkg_fromfile --skip-user ./application.flatpak
```

### URLs diretas

```bash
pkg_fromurl "https://example.com/package.deb"
```

Modos portáveis:

```bash
pkg_fromurl --tar "https://example.com/application.tar.gz"
pkg_fromurl --bin "https://example.com/application"
```

### Tarballs e binários

```bash
pkg_tarball archive.tar.gz
pkg_binary ./application
```

### Compilação a partir do código-fonte

```bash
pkg_make "https://example.com/project.git"
pkg_make --command "make install-user" "https://example.com/project.git"
```

`pkg_make` também aceita tarballs de release, URLs diretas, dependências de compilação e modo de desinstalação.

### Releases

```bash
pkg_fromrelease "https://github.com/example/project"
```

### AppImages

```bash
pkg_appimage ./Application.AppImage
```

Remoção:

```bash
pkg_appimage_rm ...
```

### Ecossistemas JavaScript

```bash
pkg_npm package-name
pkg_bun package-name
```

## Operações systemd

Serviços do sistema:

```bash
sysd_enable example.service
sysd_start example.service
sysd_stop example.service
sysd_disable example.service
```

Serviços do usuário:

```bash
sysd_enable_usr example.service
sysd_start_usr example.service
sysd_stop_usr example.service
sysd_disable_usr example.service
```

Os helpers recarregam o daemon quando necessário e registram as alterações.

## Boot e initramfs

### Atualizar bootloader

```bash
bootloader_upd
```

### Atualizar initramfs

```bash
initramfs_upd
```

### Argumentos do kernel

rpm-ostree:

```bash
kargs_upd "argument=value"
```

`grubby`:

```bash
grubbyargs_upd "argument=value"
```

### Secure Boot

```bash
secureboot_check
```

Fluxo MOK semelhante ao Ubuntu no Debian:

```bash
secureboot_check --ubuntumok
```

## Helpers diversos

### Overrides Flatpak

```bash
flatpak_override user filesystem xdg-download org.example.App
```

Formato:

```text
flatpak_override ESCOPO TIPO CONFIGURAÇÃO ALVO
```

Os escopos são `user` e `system`. `fs`, `name` e `dbus` são atalhos para `filesystem`, `talk-name` e `talk-dbus`.

### Adicionar executável ao PATH

```bash
path_link /path/to/application
```

Com o ID do aplicativo LinuxToys como nome do comando:

```bash
path_link --useappid /path/to/application
```

### Atalho de desktop

```bash
desktop_shortcut "/path/to/application"
desktop_shortcut --class ExampleApp "/path/to/application"
```

O helper usa os metadados do aplicativo fornecidos pelo LinuxToys.

### Alterar shell

```bash
shell_change /usr/bin/zsh
```

### Marcadores de recursos especiais

```bash
distrobox_created container-name
swapfile_created /swapfile
rclone_mount remote /mount/point
```

## Biblioteca de helpers

`helpers.bash` contém utilitários de configuração e detecção mais específicos:

```bash
fetch_from_mirror
multilib_chk
clinfo_chk
enable_debian_nonfree
enable_debian_backports
```

Também existem wrappers de compatibilidade como `rpmfusion_chk`, `pip_lib` e `flatpak_in_lib`, que delegam aos procedimentos correspondentes do LinuxToys.

## Biblioteca de otimizações

`optimizers.bash` contém integrações especializadas. O helper nativo atual é:

```bash
nvidia_ctkpatch
```

Também permanecem wrappers de compatibilidade:

```bash
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

Em código novo, prefira `call_script` com o procedimento subjacente quando isso tornar a dependência mais clara.

## Rastreamento de transações e reversão

Um dos principais motivos para usar as bibliotecas em vez de comandos shell brutos é o rastreamento de transações.

Helpers como `prep_edit`, `prep_create`, `pkg_install`, operações de serviços e operações de boot registram alterações no mapa de transações. Em uma falha, ou quando o usuário remove um recurso rastreado pelo LinuxToys, esses registros podem ser usados pelo sistema de reversão.

Como regra geral:

- use `prep_edit` antes de modificar um arquivo existente;
- use `prep_create` antes de criar um arquivo;
- use `prep_rm` em vez de excluir permanentemente um alvo rastreado;
- use os helpers de pacote em vez de chamar os gerenciadores diretamente;
- use `sysd_*` em vez de `systemctl` diretamente para alterações rastreadas;
- use `sudo_` em vez de `sudo`;
- reutilize os helpers do LinuxToys quando eles já representarem a operação necessária.

## Exemplo mínimo

```bash
# Nenhuma importação de biblioteca é necessária.

if ! is_debian && ! is_ubuntu; then
    die "$incompatmsg"
fi

if ! question "$msg006" "Instalar Example Service?"; then
    info "$abortmsg"
    exit 0
fi

pkg_install example-package

prep_create /etc/example/example.conf
sudo_ tee /etc/example/example.conf >/dev/null <<'EOF'
enabled=true
EOF

sysd_enable example.service
sysd_start example.service

info "$finishmsg"
```

O carregador detecta as funções referenciadas e disponibiliza automaticamente o núcleo e as bibliotecas de pacotes, sistema de arquivos e systemd necessárias.
