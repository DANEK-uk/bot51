# Використані патерни проектування / Used Design Patterns

---

## 1. Команда (Command) 

**Поведінковий патерн (Behavioral Pattern)**

**Причина використання :**  
Дозволяє інкапсулювати кожну команду користувача як окремий об’єкт. Легко додавати нові команди та змінювати їхню поведінку, не змінюючи основну логіку бота.

**Короткий опис :**  
Патерн Command інкапсулює запит як об'єкт, дозволяючи параметризувати клієнтів з різними запитами, ставити запити в чергу чи журнал, а також підтримувати відміну операцій.

**Приклад коду з проекту :**
```python
# base.py
class BotCommand(ABC):
    @abstractmethod
    def execute(self, text, chat_id, user_id):
        pass

# commands/dice.py
class DiceCommand(BotCommand):
    def __init__(self):
        self.strategy = DiceStrategy()

    def execute(self, text, chat_id, user_id):
        return self.strategy.handle(text, chat_id, user_id)
```

---

## 2. Стратегія (Strategy)

**Поведінковий патерн (Behavioral Pattern)**

**Причина використання :**  
Розділяє команду і її алгоритм (реалізацію). Дозволяє легко змінювати спосіб обробки команд без зміни коду самих команд.

**Короткий опис :**  
Патерн Strategy визначає сімейство алгоритмів, інкапсулює кожен з них та забезпечує їх взаємозамінність.

**Приклад коду з проекту :**
```python
# base.py
class CommandStrategy(ABC):
    @abstractmethod
    def handle(self, text, chat_id, user_id):
        pass

# commands/dice.py
class DiceStrategy(CommandStrategy):
    def handle(self, text, chat_id, user_id):
        return f"🎲 You rolled: {random.randint(1, 6)}"
```

---

## 3. Одинак (Singleton) 

**Твірний патерн (Creational Pattern)**

**Причина використання :**  
Забезпечує існування тільки одного екземпляра бота TelegramBot у всій програмі, уникає дублювання state і дублюючих підключень.

**Короткий опис :**  
Патерн Singleton гарантує, що клас має лише один екземпляр, і надає глобальну точку доступу до цього екземпляра.

**Приклад коду з проекту :**
```python
# core.py
class TelegramBot:
    _instance = None

    def __new__(cls, token):
        if not cls._instance:
            cls._instance = super().__new__(cls)
        return cls._instance
```

---

## 4. Фабрика (Factory) 

**Твірний патерн (Creational Pattern)**

**Причина використання :**  
Забезпечує гнучке створення нових команд за їх іменем. Додає легку масштабованість для майбутніх модулів.

**Короткий опис :**  
Патерн Factory Method визначає інтерфейс для створення об’єкта, але дозволяє підкласам змінювати тип створюваного об’єкта.

**Приклад коду з проекту :**
```python
# factories.py
class CommandFactory:
    commands_map = {
        "/help": HelpMenuCommand,
        "/dice": DiceCommand,
        # ...
    }

    @staticmethod
    def create_command(command_name):
        CommandClass = CommandFactory.commands_map.get(command_name)
        if CommandClass:
            return CommandClass()
        return None
```

---

## 5. Декоратор (Decorator) 

**Структурний патерн (Structural Pattern)**

**Причина використання :**  
Додає додаткову функціональність (логування, авторизацію) до методів обробки команд без зміни їхнього коду.

**Короткий опис :**  
Патерн Decorator дозволяє динамічно додавати об’єкти новими можливостями, обгортаючи їх у декоратори.

**Приклад коду з проекту :**
```python
# decorators.py
def log_command(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        text, chat_id, user_id = args[1:4]
        print(f"[LOG] Executing command for user {user_id} in chat {chat_id}: {text}")
        return func(*args, **kwargs)
    return wrapper

# core.py
@log_command
def handle_message(self, text, chat_id, user_id):
    ...
```

---

## 6. Ланцюг обов’язків (Chain of Responsibility) 

**Поведінковий патерн (Behavioral Pattern)**

**Причина використання :**  
Дозволяє будувати ланцюг обробників для фільтрації, цензури, логування чи додаткової обробки запитів користувача.

**Короткий опис :**  
Патерн Chain of Responsibility дозволяє передавати запит вздовж ланцюга обробників, поки хтось не обробить цей запит.

**Приклад коду з проекту :**
```python
# handlers.py
class Handler:
    def __init__(self):
        self._next_handler = None

    def set_next(self, handler):
        self._next_handler = handler
        return handler

    def handle(self, text, chat_id, user_id):
        if self._next_handler:
            return self._next_handler.handle(text, chat_id, user_id)
        return None

class CensorshipHandler(Handler):
    def handle(self, text, chat_id, user_id):
        if "badword" in text:
            return "Message blocked due to bad language!"
        return super().handle(text, chat_id, user_id)
```

---

## Загальний вигляд впровадження 

У проекті кожен патерн:
- Чітко відокремлений у своєму файлі та місці архітектури.
- Має коментар з позначенням патерна.
- Демонструє реальну користь для масштабування, підтримки та розширення логіки.
