# ВЫ НЕ ФОРКАЕТЕ ЭТОТ РЕПОЗИТОРИЙ

## ВЫ НЕ ХОСТИТЕ ЭТУ РАБОТУ

# Сборка Android-приложения

Ваша задача взять приложение Marketplace. Форкнуть репозиторий с __готовым__ заданием. Создать форк с названием `mobdev-build-lab`. Собрать там приложение, в __корень__ переместить APK и добавить его в коммит c пулл-реквестом.

## 1. Установка системных зависимостей и Android SDK

### 1.1. Установите Java 21 и необходимые утилиты

Выполните в терминале Codespaces:

```bash
sudo apt-get update
sudo apt-get install -y openjdk-21-jdk wget unzip
```

### 1.2. Настройте Android SDK

Задайте путь к Android SDK и создайте временную директорию для загрузки инструментов:

```bash
export ANDROID_HOME="$HOME/android-sdk"
mkdir -p "$ANDROID_HOME/cmdline-tools"

TMP_DIR="$(mktemp -d)"
trap 'rm -rf "$TMP_DIR"' EXIT
```

### 1.3. Скачайте Android Command-line Tools

Выполните команду:

```bash
wget -q \
  https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip \
  -O "$TMP_DIR/commandlinetools.zip"

unzip -q "$TMP_DIR/commandlinetools.zip" -d "$TMP_DIR"
```

### 1.4. Разместите инструменты в правильной директории

Создайте директорию `latest` и скопируйте в неё загруженные инструменты:

```bash
mkdir -p "$ANDROID_HOME/cmdline-tools/latest"

cp -a "$TMP_DIR/cmdline-tools/." \
  "$ANDROID_HOME/cmdline-tools/latest/"
```

### 1.5. Настройте переменные окружения

Задайте переменные окружения для текущего терминала:

```bash
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export ANDROID_HOME="$HOME/android-sdk"
export PATH="$JAVA_HOME/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH"
```

Проверьте, что Java установлена корректно:

```bash
java -version
```

Если команда не выполняется, проверьте путь к установленной версии Java:

```bash
ls -d /usr/lib/jvm/java-21-openjdk-amd64
```

### 1.6. Сохраните переменные окружения для будущих терминалов

Добавьте настройки в файл `~/.bashrc`, чтобы не задавать переменные вручную при каждом открытии терминала:

```bash
touch "$HOME/.bashrc"

if ! grep -qF '# Android SDK and Java 21' "$HOME/.bashrc"; then
  cat >> "$HOME/.bashrc" <<'EOF'

# Android SDK and Java 21
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
export ANDROID_HOME="$HOME/android-sdk"
export PATH="$JAVA_HOME/bin:$ANDROID_HOME/cmdline-tools/latest/bin:$ANDROID_HOME/platform-tools:$PATH"
EOF
fi
```

Загрузите настройки в текущий терминал:

```bash
source "$HOME/.bashrc"
```

### 1.7. Установите компоненты Android SDK

Установите Android SDK Platform Tools, Android API 34 и Build Tools 34.0.0:

```bash
sdkmanager --sdk_root="$ANDROID_HOME" \
  "platform-tools" \
  "platforms;android-34" \
  "build-tools;34.0.0"
```

### 1.8. Примите лицензии Android SDK

Выполните команду:

```bash
yes | sdkmanager --sdk_root="$ANDROID_HOME" --licenses
```

### 1.9. Проверьте установленное окружение

Выполните команды:

```bash
java -version
sdkmanager --sdk_root="$ANDROID_HOME" --list_installed

echo "Android SDK setup completed."
```

## 2. Проверьте окружение

Если вы открыли новый терминал, загрузите переменные окружения:

```bash
source "$HOME/.bashrc"
```

## 3. Установите зависимости проекта

Перейдите в корневую директорию проекта — туда, где находится файл `package.json`.

Если зависимости ещё не установлены, выполните:

```bash
npm install
```

## 4. Установите и настройте Capacitor

Если Capacitor ещё не подключён к проекту, установите необходимые пакеты:

```bash
npm install @capacitor/core @capacitor/android
npm install -D @capacitor/cli
```

Если пакеты Capacitor уже установлены и их версии согласованы, пропустите этот шаг.

### 4.1. Инициализируйте Capacitor

Если проект ещё не содержит конфигурацию Capacitor, выполните:

```bash
npx cap init
```

Укажите название приложения и идентификатор пакета.

Например:

- Название: `My Application`
- Идентификатор: `com.example.myapplication`

Используйте собственный уникальный идентификатор приложения в формате обратного доменного имени.

Если файл `capacitor.config.ts`, `capacitor.config.json` или `capacitor.config.js` уже существует, не выполняйте повторную инициализацию. Проверьте существующую конфигурацию.

### 4.2. Проверьте конфигурацию Capacitor

Убедитесь, что параметр `webDir` указывает на директорию, в которую ваш проект собирает веб-приложение.

Например, для проекта, собирающего файлы в `dist`, конфигурация может выглядеть так:

```ts
import type { CapacitorConfig } from '@capacitor/cli';

const config: CapacitorConfig = {
  appId: 'com.example.myapplication',
  appName: 'My Application',
  webDir: 'dist',
};

export default config;
```

Не заменяйте существующую конфигурацию этим примером без проверки: используйте фактические значения `appId`, `appName` и `webDir` вашего проекта.

## 5. Добавьте Android-платформу

Если Android-проект ещё не создан, выполните в корне проекта:

```bash
npx cap add android
```

Эта команда создаст директорию `android` с нативным Android-проектом.

Если директория `android` уже существует, пропустите этот шаг.

## 6. Соберите фронтенд

В корне проекта выполните:

```bash
npm run build
```

Дождитесь завершения сборки без ошибок.

Убедитесь, что результат сборки появился в директории, указанной в `webDir` файла конфигурации Capacitor. Например, если указано `dist`, проверьте наличие этой директории:

```bash
ls -la dist
```

Если проект использует другую директорию сборки, замените `dist` на соответствующий путь.

## 7. Синхронизируйте проект с Android

После успешной сборки фронтенда выполните в корне проекта:

```bash
npx cap sync android
```

Команда скопирует веб-файлы в Android-проект и синхронизирует зависимости нативных плагинов Capacitor.

Перед переходом к следующему шагу убедитесь, что команда завершилась без ошибок.

## 8. Соберите APK

Перейдите в директорию Android-проекта:

```bash
cd android
```

При необходимости предоставьте файлу Gradle Wrapper право на выполнение:

```bash
chmod +x gradlew
```

Запустите сборку отладочного APK:

```bash
./gradlew assembleDebug
```

Дождитесь завершения сборки. Если Gradle завершил работу без ошибок, APK будет создан по следующему пути:

```text
android/app/build/outputs/apk/debug/app-debug.apk
```

## 9. Скопируйте APK в корень проекта

Если сборка выполнена успешно, вернитесь в корень проекта и скопируйте туда APK.

После выполнения команды файл `app-debug.apk` появится в корне проекта.

## 10. Проверьте результат

Готовый файл:

```text
app-debug.apk
```

