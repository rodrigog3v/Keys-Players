# Keys Players — protótipo de desenvolvimento (v0.4 de especificação)

Projeto C++/JUCE para Windows 10/11 x64, com alvos **VST3** e **Standalone**. A primeira etapa usa FluidSynth para reproduzir arquivos **SF2** recebendo MIDI do host. Também identifica o cabeçalho `CBIN` de um `.npno`, mas **ainda não decodifica nem reproduz áudio NPno**.

> Estado honesto do projeto: é código-fonte inicial, ainda não um instalador pronto. Não foi compilado/testado neste ambiente Windows. O leitor NPno e a seleção MIDI direta no modo Standalone são trabalho pendente. No Ableton Live, o MIDI do controlador deve chegar ao plugin pela faixa MIDI.

## Funcionalidades implementadas no código inicial

- Projeto JUCE CMake com alvos VST3 e Standalone.
- Entrada MIDI do host: note on/off, velocity, sustain/CC, pitch bend, aftertouch e program change.
- Carregamento de um banco SF2 pelo botão `Abrir SF2`.
- Processamento SF2 via FluidSynth, com reverb e chorus desativados por padrão para facilitar comparação neutra.
- Botão `Inspecionar NPno` para validar a assinatura `CBIN` e deixar explícito que a reprodução NPno ainda não está implementada.
- Armazenamento do caminho do SF2 no estado do plugin; o arquivo continua no caminho original e precisa continuar acessível.

## Requisitos no Windows

1. **Visual Studio 2022 Community** com a carga de trabalho “Desenvolvimento para desktop com C++”.
2. **Git for Windows**.
3. **CMake 3.22+**.
4. **vcpkg** configurado para Visual Studio e a dependência FluidSynth instalada.
5. Conexão com a internet na primeira compilação, pois CMake baixa o JUCE do repositório oficial.

### Instalar dependência FluidSynth

Abra PowerShell e execute (ajuste o caminho se instalou o vcpkg em outro local):

```powershell
git clone https://github.com/microsoft/vcpkg C:\dev\vcpkg
C:\dev\vcpkg\bootstrap-vcpkg.bat
C:\dev\vcpkg\vcpkg.exe install fluidsynth:x64-windows
```

### Compilar VST3 e Standalone

Extraia a pasta do projeto, abra PowerShell nela e execute:

```powershell
cmake -S . -B build -A x64 `
  -DCMAKE_TOOLCHAIN_FILE=C:/dev/vcpkg/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release --target KeysPlayers_VST3 KeysPlayers_Standalone
```

Os nomes e caminhos finais dos artefatos podem variar um pouco conforme a versão do JUCE/CMake. Procure os diretórios `build/KeysPlayers_artefacts/Release/VST3` e `build/KeysPlayers_artefacts/Release/Standalone`.

Se CMake não encontrar o pacote FluidSynth, confirme que o `vcpkg install` foi executado para `x64-windows` e que o caminho do toolchain está correto.

## Testar no Ableton Live

1. Compile o alvo VST3.
2. Copie a pasta `Keys Players.vst3` para `C:\Program Files\Common Files\VST3` (ou configure uma pasta VST3 personalizada).
3. Abra o Ableton Live e faça uma nova busca/rescan de plugins.
4. Crie uma faixa MIDI e carregue `Keys Players`.
5. Selecione a faixa, escolha o controlador MIDI como entrada, arme a faixa e monitore em `Auto` ou `In`.
6. Abra o plugin e clique em **Abrir SF2** para selecionar seu arquivo `.sf2`.
7. Para tocar ao vivo com baixa latência, selecione seu driver ASIO em Preferences → Audio no Ableton.

O plugin recebe MIDI encaminhado pelo Ableton. A configuração do dispositivo de áudio/ASIO fica a cargo do host quando executado como plugin.

## Standalone

Execute o artefato `Keys Players.exe`. Selecione o driver e o dispositivo de áudio na janela de configuração de áudio do wrapper JUCE, se apresentada. O suporte à seleção direta de portas MIDI no Standalone ainda precisa ser concluído/validado; para esta primeira versão, a rota confiável de teste é pelo Ableton Live.

## Plano para NPno

O arquivo `.npno` não é SF2 e não pode ser enviado diretamente ao FluidSynth. A rotina `inspectNordPianoFile()` somente faz uma inspeção inicial do cabeçalho. A próxima etapa é integrar/portar um decodificador compatível com a versão do formato do arquivo, mapear as regiões e camadas, extrair as amostras sem perdas e implementar reprodução com loops e velocity layers. Até isso ser feito, o programa **não toca arquivos `.npno`**.

## Fidelidade

O SF2 é reproduzido pelo FluidSynth, portanto o resultado depende do preset/banco e do motor SF2. Para NPno, fidelidade total não pode ser prometida antes de confirmar que todas as amostras, loops, camadas e metadados são extraídos e reproduzidos corretamente. O projeto não converte SF2 para MP3 nem altera os arquivos de origem.

## Licenças e marcas

JUCE e FluidSynth têm licenças próprias; revise-as antes de distribuir o produto. Nord/Clavia são marcas de terceiros; este projeto não é afiliado nem endossado pela Clavia.

## Interface v0.2 — workstation visual prototype

A interface foi ampliada para uma workstation inspirada na referência do usuário: seis painéis de layer, controles M/S visuais, sliders, oito pads, master, ações de Live Set, MIDI Learn, gravação e seleção de arquivos SF2/Nord.

**Limitação explícita desta etapa:** o motor de áudio continua com uma única instância SF2 ativa. Os seis layers, pads, MIDI Learn, Live Set, gravação e efeitos ainda são principalmente estrutura visual/controles de preparação e não devem ser considerados recursos de áudio concluídos. O botão Nord inspeciona o `.npno`, mas não reproduz seu áudio. A entrada MIDI no VST3 é fornecida pelo host; a seleção direta de dispositivos MIDI e ASIO no Standalone ainda precisa de integração e testes.

Para compilar, siga os requisitos da seção anterior. Esta revisão não foi compilada neste ambiente; valide no Visual Studio antes de usar em produção.


## Especificação expandida v0.4

Consulte `REQUIREMENTS_V0.4.md` para a lista consolidada das funções observadas nas imagens: editor de camada, zonas/split, velocity, MIDI Learn, pads, efeitos, Live Sets, gravação e critérios de aceite. Este documento é o plano de implementação; ele não significa que esses recursos já estejam operacionais.

## Compilação automática do instalador (GitHub Actions)

O repositório inclui `.github/workflows/windows-installer.yml` e `installer.iss` para tentar compilar o aplicativo e gerar `KeysPlayers-Setup.exe` em um runner Windows hospedado pelo GitHub. Esta automação ainda precisa ser executada e o resultado precisa passar por testes; ela **não é prova de que a compilação terá sucesso**, nem torna os recursos incompletos concluídos.

Para disparar a compilação sem instalar ferramentas no seu PC:

1. Crie um repositório no GitHub e envie o conteúdo desta pasta para ele (ou peça a alguém de confiança para fazer essa configuração inicial).
2. No repositório, abra **Actions** e habilite workflows se o GitHub solicitar.
3. Selecione **Build Keys Players Windows installer** e clique em **Run workflow**.
4. Se a execução terminar com sucesso, abra o resumo da execução e baixe o artefato `Keys-Players-Windows-Installer`.
5. Teste o instalador em Windows antes de usar em apresentação ao vivo.

Se o workflow falhar, consulte a primeira etapa vermelha nos logs. Pode ser necessário corrigir incompatibilidades de build ou empacotamento. A compilação depende de acesso à internet para baixar JUCE e FluidSynth.

**Limitações funcionais desta versão:** SF2 é a única rota de reprodução prevista; arquivos Nord `.npno` apenas passam por uma inspeção inicial e não são reproduzidos. As seis camadas, pads, MIDI Learn, Live Sets, gravação e efeitos avançados ainda não estão implementados como funções completas. O suporte a MIDI no Standalone também precisa ser validado.
