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

    User --> UC1
    User --> UC3
    User --> UC4
    
    UC1 -. "<<include>>" .-> UC2
    Server --> UC2
    
    UC4 -. "<<include>>" .-> UC5
```
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

    UserModel "1" -- "*" MediaMessage : відправляє
    UserModel "2" -- "*" StreakSession : підтримують
    UserModel "1" -- "1" DynamicConfig : має налаштування
```
