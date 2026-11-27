# Fingprint SDK для Android

Fingprint SDK получает идентификатор посетителя через Fingprint API.
Для интеграции нужны Android 6.0 (API 23) или новее и публичный API-ключ Fingprint.
SDK поддерживает Kotlin и Java.

[Документация Android SDK](https://docs.fingprint.com/docs/integrations/android).

## Установка

Добавьте Maven-репозиторий в `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://cdn.fingprint.com/android/maven/") }
    }
}
```

Добавьте зависимость в `build.gradle.kts` модуля приложения:

```kotlin
dependencies {
    implementation("ru.fingerru:agent:1.2.0")
}
```

## Получение идентификатора

`VisitorResult.user` — идентификатор посетителя. `requestId` — идентификатор
запроса; он может отсутствовать.

В примерах замените `PUBLIC_KEY` на свой публичный API-ключ.

### Kotlin

Вызовите `get()` из корутины. При отмене корутины SDK прекращает ожидание ответа.

```kotlin
import android.content.Context
import ru.fingerru.agent.Configuration
import ru.fingerru.agent.FingprintAgent
import ru.fingerru.agent.VisitorResult

suspend fun identifyVisitor(context: Context): VisitorResult {
    val agent = FingprintAgent(context, Configuration(apiKey = "PUBLIC_KEY"))
    return agent.get()
}
```

### Java

`getAsync()` вызывает `onSuccess` или `onError` на главном потоке.
Если экран закрывается до получения ответа, вызовите `cancel()` у возвращённого
`Cancellable`: ещё не начавшийся колбэк не будет вызван.

```java
import android.content.Context;
import android.util.Log;
import ru.fingerru.agent.Callback;
import ru.fingerru.agent.Cancellable;
import ru.fingerru.agent.Configuration;
import ru.fingerru.agent.FingprintAgent;
import ru.fingerru.agent.FingprintAgentException;
import ru.fingerru.agent.VisitorResult;

public final class VisitorIdentification {
    public static Cancellable identifyVisitor(Context context) {
        FingprintAgent agent = new FingprintAgent(context, new Configuration("PUBLIC_KEY"));
        return agent.getAsync(new Callback<VisitorResult>() {
            @Override
            public void onSuccess(VisitorResult result) {
                String visitorId = result.getUser();
                // Используйте visitorId в приложении.
            }

            @Override
            public void onError(FingprintAgentException error) {
                Log.w("Fingprint", "Ошибка идентификации: " + error.getCode());
            }
        });
    }
}
```

Одновременные вызовы с одинаковыми настройками используют один запрос.
Отмена одного вызова не отменяет остальные.

## Настройки и ошибки

- **Геолокация.** По умолчанию отключена. Чтобы включить её, задайте
  `allowUseOfLocationData = true`. Приложение само объявляет и запрашивает
  `ACCESS_COARSE_LOCATION` или `ACCESS_FINE_LOCATION`. Приблизительной геолокации
  достаточно. Остальные необходимые разрешения добавляются при объединении манифестов.
- **Тайм-аут HTTP.** По умолчанию 10 секунд. Например, для 15 секунд используйте
  `get(timeoutMillis = 15_000)` в Kotlin или `getAsync(15_000L, callback)` в Java.
  SDK не повторяет запрос автоматически.
- **Ошибки.** В Kotlin обработайте `FingprintAgentException`, в Java — `onError`.
  Причину определяйте по `code` (`getCode()` в Java). Только `HTTP_ERROR` содержит
  `httpStatus`. Другие коды: `INVALID_CONFIGURATION`, `COLLECTION_FAILED`,
  `PAYLOAD_FAILED`, `NETWORK_FAILED`, `TIMEOUT`, `INVALID_RESPONSE`.

[История изменений](CHANGELOG.md)
