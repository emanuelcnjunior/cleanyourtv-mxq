# CleanYourTV MXQ — instalar e compilar no Windows

Versão do launcher CleanYourTV adaptada para a **MXQ Pro 4K 5G com Android 7.0 (API 24)**. Este repositório guarda o **código-fonte**, não um instalador pronto. O aplicativo usa o pacote `com.gathin.tvhome`.

> **Atenção:** nem toda TV Box chamada “MXQ Pro 4K 5G” usa o mesmo hardware ou firmware. Confirme a versão do Android e faça um teste antes de configurá-lo como launcher padrão. Mantenha o launcher original instalado e habilitado.

## 1. Instalar as ferramentas no PC

No Windows, instale:

1. **Git:** https://git-scm.com/download/win
2. **JDK 17:** https://adoptium.net/temurin/releases/?version=17
3. **Android Studio:** https://developer.android.com/studio
4. **Android SDK Platform-Tools (ADB):** pelo SDK Manager do Android Studio, ou em https://developer.android.com/tools/releases/platform-tools

No **Android Studio > SDK Manager**, instale:
- Android SDK Platform **34**
- Android SDK Build-Tools **34.x**
- Android SDK Platform-Tools
- Android SDK Command-line Tools (latest)

Anote a pasta do SDK. Normalmente:
```text
C:\Users\SEU_USUARIO\AppData\Local\Android\Sdk
```

Abra o **CMD** e verifique:
```bat
git --version
java -version
adb version
```

`java -version` deve exibir **17**. Se outra versão estiver ativa, configure o JDK 17 nesta janela do CMD (ajuste a pasta para a versão instalada):

```bat
set "JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-17.0.20.101-hotspot"
set "PATH=%JAVA_HOME%\bin;%PATH%"
java -version
```

Se `adb` não for reconhecido, use o `adb.exe` na pasta `platform-tools` do SDK ou adicione essa pasta ao `PATH`.

## 2. Baixar o projeto

No CMD:
```bat
cd /d C:\
git clone https://github.com/emanuelcnjunior/cleanyourtv-mxq.git
cd C:\cleanyourtv-mxq\launcher
```

O projeto Android fica em **`launcher\`**; não execute o Gradle na raiz do repositório.

## 3. Informar o caminho do Android SDK

Dentro de `C:\cleanyourtv-mxq\launcher`, crie `local.properties`:

```bat
notepad local.properties
```

Adicione uma linha (ajuste seu usuário e use barras normais):

```properties
sdk.dir=C:/Users/SEU_USUARIO/AppData/Local/Android/Sdk
```

Salve. Esse arquivo é local e não deve ser enviado ao GitHub.

## 4. Compilar o APK

No CMD, ainda em `C:\cleanyourtv-mxq\launcher`:

```bat
gradlew.bat clean assembleDebug
```

Na primeira execução, o Gradle baixa dependências; é necessária conexão à internet.

Quando aparecer `BUILD SUCCESSFUL`, o APK estará em:

```text
C:\cleanyourtv-mxq\launcher\app\build\outputs\apk\debug\app-debug.apk
```

O projeto usa **Gradle 8.9**, **Android Gradle Plugin 8.5.2**, `compileSdk 34`, `minSdk 24` e Java 17.

## 5. Preparar a outra MXQ

Na TV Box:

1. Abra **Configurações > Sobre** e habilite as **Opções do desenvolvedor** (normalmente tocando várias vezes em “Número da versão”).
2. Ative **Depuração USB** ou **Depuração ADB**. Em alguns firmwares, ADB via rede precisa ser ativado separadamente.
3. Conecte a TV Box e o computador na mesma rede.
4. Descubra o **IP da nova MXQ**. **Não reutilize automaticamente o IP da primeira caixa.**

No CMD, teste (substitua pelo IP real):

```bat
adb connect IP_DA_MXQ:5555
adb devices -l
adb -s IP_DA_MXQ:5555 shell getprop ro.build.version.release
adb -s IP_DA_MXQ:5555 shell getprop ro.build.version.sdk
```

Confirme que aparece `device`, não `offline`, e que o SDK do Android é **24 ou superior**. Se a porta 5555 não estiver habilitada, configure primeiro a depuração ADB no aparelho ou conecte por USB; não presuma que toda MXQ aceita ADB via Wi-Fi.

## 6. Instalar o CleanYourTV

Na pasta `launcher`:

```bat
adb -s IP_DA_MXQ:5555 install --no-streaming -r app\build\outputs\apk\debug\app-debug.apk
```

Se `--no-streaming` não for reconhecido pela versão do ADB, tente `install -r`.

Confirme:

```bat
adb -s IP_DA_MXQ:5555 shell pm path com.gathin.tvhome
```

Depois, abra o CleanYourTV na TV e teste a navegação e o controle remoto.

> Se aparecer `INSTALL_FAILED_UPDATE_INCOMPATIBLE`, pode haver uma instalação com assinatura diferente. **Não desinstale sem antes avaliar os dados e o launcher de recuperação**.

## 7. Definir como launcher padrão

Na MXQ, pressione **HOME**, selecione **Início / CleanYourTV** e escolha **Sempre**. Teste abrindo um aplicativo e retornando com HOME.

Confirme pelo CMD:

```bat
adb -s IP_DA_MXQ:5555 shell cmd package resolve-activity --brief -a android.intent.action.MAIN -c android.intent.category.HOME
```

Resultado esperado:

```text
com.gathin.tvhome/.MainActivity
```

**Não desinstale nem desative o MediaBox Launcher original** (`com.droidlogic.mboxlauncher`). Ele é a alternativa de recuperação.

## 8. Conferir e solucionar problemas

```bat
adb -s IP_DA_MXQ:5555 shell pidof com.gathin.tvhome
adb -s IP_DA_MXQ:5555 shell pm list packages -3
adb -s IP_DA_MXQ:5555 shell cat /proc/meminfo
```

Se o ADB ficar `offline`:

```bat
adb disconnect IP_DA_MXQ:5555
adb kill-server
adb start-server
adb connect IP_DA_MXQ:5555
adb devices -l
```

Se o launcher não iniciar, **não desabilite o launcher original**. Para abrir o MediaBox via ADB, caso esteja disponível:

```bat
adb -s IP_DA_MXQ:5555 shell am start -n com.droidlogic.mboxlauncher/.Launcher
```

Caso o problema seja somente o padrão HOME, altere a preferência pelas Configurações do Android.

## 9. Atualizar o código e recompilar

```bat
cd /d C:\cleanyourtv-mxq
git pull
cd launcher
gradlew.bat assembleDebug
```

Depois reinstale o APK com `adb install -r`, conforme a seção 6.

## Importante: o que este projeto **não** faz automaticamente

Compilar e instalar o launcher **não reproduz a limpeza de aplicativos, a desativação de serviços ou os ajustes de animação** feitos manualmente na primeira MXQ. Esses ajustes são específicos do firmware e devem ser verificados separadamente em cada aparelho.
