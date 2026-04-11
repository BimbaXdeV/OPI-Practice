flowchart LR
    %% Актори
    User((Користувач))
    Server((Сервер\nавторизації))

    %% Межа системи
    subgraph System [Додаток Streaker]
        UC1([OPT-3: Вікно реєстрації])
        UC2([OPT-4: Верифікація користувача])
        UC3([OPT-11: Зовнішній вид профілю])
        UC4([Надіслати фото-стрік])
        UC5([Оновити віджет на екрані])
    end

    %% Зв'язки
    User --> UC1
    User --> UC3
    User --> UC4
    
    UC1 -. "<<include>>" .-> UC2
    Server --> UC2
    
    UC4 -. "<<include>>" .-> UC5

classDiagram
    %% Класи на основі OPT-5
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

    %% Зв'язки
    UserModel "1" -- "*" MediaMessage : відправляє
    UserModel "2" -- "*" StreakSession : підтримують
    UserModel "1" -- "1" DynamicConfig : має налаштування

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

    
    
