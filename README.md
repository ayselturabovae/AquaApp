# AquaApp — CI/CD для Android (МДК 01.03 ПР 10)

Готовый проект для практической работы по настройке CI/CD конвейера:
- **GitHub Actions** — автоматическая сборка при пуше
- **Fastlane** — автоматизация подписи и загрузки
- **Firebase App Distribution** — доставка сборок тестировщикам

---

## 📁 Структура проекта

```
AquaApp/
├── .github/workflows/
│   ├── android.yml          # Основной CI/CD workflow
│   └── bump-version.yml     # Автоинкремент версии (Задание Б)
├── fastlane/
│   ├── Fastfile             # Lane distribute для Firebase
│   ├── Pluginfile           # Plugin firebase_app_distribution
│   └── Gemfile              # Зависимости Ruby
├── scripts/
│   └── bump_version.sh      # Скрипт увеличения версии (Задание Б)
├── app/
│   ├── build.gradle.kts     # Настроенные buildTypes, signingConfig
│   ├── src/main/java/com/example/aquaapp/
│   │   ├── MainActivity.kt      # UI приложения (Jetpack Compose)
│   │   ├── TaskLogic.kt         # Логика (для unit-тестов)
│   │   └── ui/theme/            # Color.kt, Theme.kt, Type.kt
│   └── src/test/java/com/example/aquaapp/
│       └── TaskLogicTest.kt     # 12 unit-тестов
├── gradle/wrapper/          # Gradle wrapper
├── gradlew, gradlew.bat
├── settings.gradle.kts
├── build.gradle.kts
├── gradle.properties
├── keystore.properties.example  # Шаблон для локальных паролей
├── .gitignore               # Исключает keystore, local.properties, секреты
└── README.md                # Этот файл
```

---

## 🚀 Быстрый старт (что сделать по порядку)

### 1. Откройте проект в Android Studio
- File → Open → выберите папку `AquaApp`
- Дождитесь синхронизации Gradle
- Запустите на эмуляторе — проверьте, что приложение работает

### 2. Запустите unit-тесты (СКРИНШОТ для отчёта)
- Правый клик на `TaskLogicTest` → Run 'TaskLogicTest'
- **СКРИНШОТ**: зелёная полоса "12 tests passed" → сохраните как `image2.png`
- В отчёте: **"Рисунок 2 — Результаты выполнения unit-тестов"**

### 3. Создайте keystore.jks (СКРИНШОТ для отчёта)
```powershell
# В PowerShell в папке app/
cd app
keytool -genkeypair -alias aquaapp -keyalg RSA -keysize 2048 -validity 10000 -keystore keystore.jks
```
- Alias: `aquaapp` (запомните!)
- Пароли: придумайте два (keystore + key)
- **СКРИНШОТ**: процесс создания → `image3.png`
- В отчёте: **"Рисунок 3 — Создание keystore.jks"**

### 4. Закодируйте keystore в Base64 (для GitHub Secrets)
```powershell
[Convert]::ToBase64String([IO.File]::ReadAllBytes("app\keystore.jks")) | Set-Clipboard
```
- В буфере обмена будет длинная строка — вставьте в GitHub Secrets

### 5. Создайте `keystore.properties` ЛОКАЛЬНО (не коммитить!)
Скопируйте `keystore.properties.example` → `keystore.properties` и заполните:
```properties
KEYSTORE_PASSWORD=ваш_пароль_keystore
KEY_ALIAS=aquaapp
KEY_PASSWORD=ваш_пароль_ключа
```

### 6. Настройте Firebase Console (СКРИНШОТЫ для отчёта)
1. https://console.firebase.google.com → **Create project** → `AquaApp-CICD`
2. **Add app** → Android → package: `com.example.aquaapp`
3. **App Distribution** → **Get started** → создайте группу `qa-team` + email тестировщика
4. **Project settings** ⚙️ → **Service accounts** → **Generate new private key** → сохраните JSON
5. **App ID** (формат `1:123456789:android:abcdef`) — скопируйте для Fastfile
- **СКРИНШОТЫ**: проект в Firebase, приложение, группа тестировщиков → `image4.png`, `image5.png`
- В отчёте: **"Рисунок 4 — Приложение в Firebase"**, **"Рисунок 5 — Группа тестировщиков"**

### 7. Создайте GitHub репозиторий и добавьте секреты
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/ВАШ_ЮЗЕР/AquaApp.git
git push -u origin main
```

**Settings → Secrets and variables → Actions → New repository secret (5 шт.):**

| Name | Value |
|------|-------|
| `ANDROID_KEYSTORE_BASE64` | Base64 строка из шага 4 |
| `KEYSTORE_PASSWORD` | пароль от keystore |
| `KEY_ALIAS` | `aquaapp` |
| `KEY_PASSWORD` | пароль от ключа |
| `FIREBASE_SERVICE_ACCOUNT` | **весь JSON** из Firebase (сырой, не Base64!) |

- **СКРИНШОТ**: страница Secrets (значения скрыты) → `image6.png`
- В отчёте: **"Рисунок 6 — Секреты репозитория"**

### 8. Обновите Fastfile с вашим App ID
Откройте `fastlane/Fastfile` и замените:
```ruby
app: "1:ВАШ_FIREBASE_APP_ID:android:ВАШ_HASH",
```
на реальный App ID из Firebase.

### 9. Запушьте изменения — запустится CI/CD
```bash
git add .
git commit -m "Configure Fastlane with App ID"
git push
```

### 10. Смотрите GitHub Actions (СКРИНШОТЫ для отчёта)
- Откройте репозиторий → **Actions** → увидите запуск `Android CI/CD`
- **Первый запуск может упасть** — это нормально (часто из-за неверного JSON в секрете)
- **СКРИНШОТ упавшего билда** → `image7.png` (пункт 9 отчёта "Первый запуск")
- **СКРИНШОТ лога ошибки** → `image8.png`
- Исправьте секрет `FIREBASE_SERVICE_ACCOUNT` (вставьте сырой JSON!)
- **Re-run all jobs** → должен стать зелёным

- **СКРИНШОТ успешного запуска** → `image9.png` (пункт 10 "Успешный запуск")
- **СКРИНШОТ артефактов** (APK + AAB) → `image10.png`
- В отчёте: **"Рисунок 7 — Ошибка первого запуска"**, **"Рисунок 8 — Лог ошибки"**, **"Рисунок 9 — Успешный запуск"**, **"Рисунок 10 — Артефакты"**

### 11. Проверьте Firebase и почту тестировщика (СКРИНШОТЫ)
- Firebase Console → **App Distribution** → **Releases** → появилась сборка
- **СКРИНШОТ** → `image11.png`
- Почта тестировщика → письмо от "Firebase App Distribution"
- **СКРИНШОТ** → `image12.png`
- В отчёте: **"Рисунок 11 — Сборка в Firebase"**, **"Рисунок 12 — Письмо тестировщику"**

---

## 📋 Самостоятельная работа (обязательно для оценки!)

### Задание А: Сборка AAB (уже настроено в workflow)
- В `android.yml` есть шаг `Assemble Release AAB` и `Upload AAB Artifact`
- **СКРИНШОТ**: артефакт `app-release-aab` в Actions → `image13.png`
- В отчёте: **"Рисунок 13 — Артефакт AAB"**

### Задание Б: Автоинкремент версии
1. Создайте ветку `release/1.0`:
```bash
git checkout -b release/1.0
git push origin release/1.0
```
2. В GitHub Actions запустится workflow `Version Bump`
3. Он выполнит `scripts/bump_version.sh`:
   - `versionCode`: 1 → 2
   - `versionName`: "1.0.0" → "1.0.1"
4. Бот `github-actions` закоммитит изменения с сообщением `chore: bump version [skip ci]`

- **СКРИНШОТ**: вкладка Actions — запуск `Version Bump` → `image14.png`
- **СКРИНШОТ**: изменённый `build.gradle.kts` (новая версия) → `image15.png`
- **СКРИНШОТ**: коммит бота в истории → `image16.png`
- В отчёте: **"Рисунок 14 — Запуск version bump"**, **"Рисунок 15 — Версия в build.gradle.kts"**, **"Рисунок 16 — Коммит бота"**

### Задание В: Проверка секретов
- Убедитесь: `keystore.jks`, `keystore.properties`, `local.properties` в `.gitignore`
- Все пароли только в GitHub Secrets
- В репозитории их нет
- **СКРИНШОТ**: `.gitignore` в репозитории → `image17.png`
- В отчёте: **"Рисунок 17 — Файл .gitignore"**

---

## 📸 Полный список скриншотов для отчёта

| # | Что снимать | Где | Пункт отчёта |
|---|-------------|-----|--------------|
| 1 | Главный экран приложения | Эмулятор/телефон | 1. Создание проекта |
| 2 | Результаты unit-тестов (12 passed) | Android Studio | 2. Unit-тесты |
| 3 | Создание keystore.jks | PowerShell | 4. Секреты подписи |
| 4 | Приложение в Firebase | Firebase Console | 5. Настройка Firebase |
| 5 | Группа тестировщиков qa-team | Firebase Console | 5. Настройка Firebase |
| 6 | Секреты GitHub (5 шт., скрыты) | GitHub Settings | 6. Репозиторий и секреты |
| 7 | Упавший первый запуск | GitHub Actions | 9. Первый запуск |
| 8 | Лог ошибки Fastlane | GitHub Actions | 9. Первый запуск |
| 9 | Успешный запуск (все зелёные) | GitHub Actions | 10. Успешный запуск |
| 10 | Артефакты: APK + AAB | GitHub Actions → Artifacts | 10. Успешный запуск |
| 11 | Сборка в Firebase Releases | Firebase Console | 10. Доставка тестировщикам |
| 12 | Письмо тестировщику | Почта | 10. Доставка тестировщикам |
| 13 | Артефакт AAB (Задание А) | GitHub Actions → Artifacts | 11. Задание А |
| 14 | Запуск Version Bump | GitHub Actions | 12. Задание Б |
| 15 | build.gradle.kts с новой версией | GitHub / локально | 12. Задание Б |
| 16 | Коммит бота [skip ci] | GitHub History | 12. Задание Б |
| 17 | .gitignore в репозитории | GitHub | 13. Задание В |

---

## 📝 Структура отчёта (по примерам)

Создайте файл `ПР10_Фамилия_Группа.docx` со следующими разделами:

1. **Титульный лист** (ВУЗ, МДК 01.03, ПР 10, ФИО, группа, год)
2. **Цель работы** (из методички)
3. **Теоретическая база** (2–3 абзаца: CI, CD, GitHub Actions, Fastlane, Firebase)
4. **Ход работы** (пункты 1–14 с рисунками и подписями "Рисунок N — ...")
5. **Самостоятельная работа** (А, Б, В с рисунками)
6. **Проверка критериев оценки** (таблица)
7. **Вывод** (абзац: что сделали, что автоматизировало, ошибки и исправления)

---

## ⚠️ Типичные ошибки и решения

| Ошибка | Причина | Решение |
|--------|---------|---------|
| `service_credentials_file_content not found` | Старый параметр в методичке | Используйте `service_credentials_json_data` (уже в Fastfile) |
| Keystore not found при сборке | Decode keystore стоит ПОСЛЕ assemble | В workflow Decode Keystore **ДО** Assemble Release |
| `actions v3 deprecated` | В методичке старые версии | Используйте `@v4` (уже в workflow) |
| JDK version mismatch | Проект требует JDK 25 | В workflow `java-version: 25` (проверьте `gradle-daemon-jvm.properties`) |
| Fastlane: invalid JSON | В секрет вставлен Base64 вместо JSON | Вставьте **сырой JSON** из скачанного файла |
| Version bump не запускается | Ветка не `release/**` | Создайте ветку `release/1.0` именно так |
| Бот коммитит бесконечно | Нет `[skip ci]` в сообщении | В workflow уже есть `[skip ci]` |

---

## 🔧 Локальная проверка перед пушем

```bash
# 1. Тесты
./gradlew test

# 2. Сборка release (подпишется если есть keystore.properties)
./gradlew assembleRelease bundleRelease

# 3. Fastlane (локально, если настроены ruby/bundle)
cd ..
bundle install
bundle exec fastlane distribute
```

---

## 📞 Поддержка

Если что-то не работает:
1. Проверьте логи в GitHub Actions (нажмите на упавший шаг)
2. Сравните секреты с таблицей выше
3. Убедитесь, что `FIREBASE_SERVICE_ACCOUNT` — это **сырой JSON**, а не Base64
4. Проверьте, что App ID в Fastfile точно совпадает с Firebase

---

**Удачи с сдачей! 🎓** 
Все скриншоты делайте по мере прохождения шагов — так не забудете ничего.