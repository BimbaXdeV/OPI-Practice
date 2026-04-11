# Лабораторна робота: Проєктування архітектури застосунку

### Крок 1. Опис проєкту
**Назва проєкту:** "Streaker" 
**Опис:** Мобільний клієнт-серверний застосунок для Android, націлений на підтримання щоденного спілкування між близькими людьми (формат "стріків") за допомогою обміну фотографіями через віджет на головному екрані.

### Крок 2. Функціональні вимоги
* **FR-01 (Реєстрація):** Система повинна надавати інтерфейс для реєстрації користувача та проводити його верифікацію (задачі OPT-3, OPT-4).
* **FR-02 (Профіль):** Користувач повинен мати можливість налаштувати зовнішній вигляд свого профілю (задача OPT-11).
* **FR-03 (Мережева комунікація):** Система повинна передавати медіафайли між клієнтом та сервером за визначеним мережевим протоколом (задача OPT-7).
* **FR-04 (База даних):** Серверна частина повинна зберігати користувачів, повідомлення та стан "стріків" у реляційній базі даних (задача OPT-5).
* **FR-05 (Динамічна конфігурація):** Система повинна автоматично оновлювати стан віджета з використанням анімацій при надходженні нових даних (задачі OPT-13, OPT-14).

### Крок 3. Діаграма прецедентів (Use Case Diagram)


```mermaid
flowchart LR
    User((Користувач))
    Server((Сервер\nавторизації))

    subgraph System [Додаток Streaker]
        UC1([OPT-3: Вікно реєстрації])
        UC2([OPT-4: Верифікація користувача])
        UC3([OPT-11: Зовнішній вид профілю])
        UC4([Надіслати фото-стрік])
        UC5([Оновити віджет на екрані])
    end

    %% Робимо фон прозорим, але залишаємо рамку системи
    style System fill:transparent,stroke:#666,stroke-width:2px,stroke-dasharray: 5 5

    User --> UC1
    User --> UC3
    User --> UC4
    
    UC1 -. "<<include>>" .-> UC2
    Server --> UC2
    
    UC4 -. "<<include>>" .-> UC5
```
### Крок 4. Діаграма класів (Class Diagram)

```mermaid
classDiagram
    class UserModel {
        +UUID userId
        +String phoneNumber
        +Boolean isVerified
        +OPT_4_VerifyUser() Boolean
        +OPT_11_UpdateProfile() void
    }

    class MediaMessage {
        +UUID messageId
        +UUID senderId
        +String imageUrl
        +DateTime timestamp
        +OPT_7_NetworkSend() Boolean
    }

    class StreakSession {
        +UUID sessionId
        +UUID user1_Id
        +UUID user2_Id
        +int currentDays
        +DateTime lastInteraction
        +incrementStreak() void
        +resetStreak() void
    }

    class DynamicConfig {
        +String animationType
        +String themeColor
        +OPT_14_ApplyConfig() void
        +OPT_13_PlayAnimation() void
    }

    class NotificationService {
        +String pushToken
        +sendStreakWarning(UUID userId) void
        +notifyNewMessage(UUID receiverId) void
    }

    UserModel "1" -- "*" MediaMessage : відправляє
    UserModel "2" -- "*" StreakSession : підтримують
    UserModel "1" -- "1" DynamicConfig : має налаштування
    UserModel "1" -- "1" NotificationService : отримує сповіщення
```
### Крок 5. Діаграма послідовності (Sequence Diagram)

```mermaid
sequenceDiagram
    actor U as Користувач
    participant UI as Клієнт (UI)
    participant Net as OPT-7 Мережевий Протокол
    participant Srv as OPT-8 Сервер (Бекенд)
    participant DB as OPT-5 База Даних

    U->>UI: Робить фото для стріку
    activate UI
    UI->>UI: OPT-13 Відтворення анімації
    UI->>Net: sendMedia(userId, file)
    activate Net
    Net->>Srv: POST /api/v1/streak/media
    activate Srv
    
    Srv->>DB: saveMessage(file_url, timestamp)
    activate DB
    DB-->>Srv: success (messageId)
    
    Srv->>DB: checkStreakStatus(userId)
    DB-->>Srv: streak_updated
    deactivate DB
    
    Srv-->>Net: 200 OK (упішно надіслано)
    deactivate Srv
    Net-->>UI: Відобразити статус доставки
    deactivate Net
    
    UI-->>U: Стрік подовжено!
    deactivate UI
```

### Крок 6. Матриця трасовності

| ID вимоги | Задачі в Jira | Прецедент (Use Case) | Задіяні класи (OPT-5) | Діаграма послідовності |
| :--- | :--- | :--- | :--- | :--- |
| **FR-01** | OPT-3, OPT-4 | Вікно реєстрації, Верифікація | `UserModel` | Ні |
| **FR-02** | OPT-11 | Зовнішній вид профілю | `UserModel` | Ні |
| **FR-03** | OPT-7, OPT-8 | Надіслати фото-стрік | `MediaMessage` | **Так (OPT-7)** |
| **FR-04** | OPT-5 | Усі (Основа системи) | `StreakSession`, `UserModel` | **Так (DB збереження)** |
| **FR-05** | OPT-13, OPT-14 | Оновити віджет на екрані | `DynamicConfig` | Ні |
