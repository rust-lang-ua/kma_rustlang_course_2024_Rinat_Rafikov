# Практична робота: Серіалізація та десеріалізація з `serde`

### Підготовка

1. **Налаштування проєкту**  
   Створіть новий проєкт у Rust:

   ```bash
   cargo new serde_practice --bin
   cd serde_practice
   ```

2. **Додайте залежності в `Cargo.toml`**
   ```toml
   [dependencies]
   serde = { version = "1.0", features = ["derive"] }
   serde_json = "1.0"
   serde_yaml = "0.8"
   toml = "0.5"
   ```

---

### 1. Базова серіалізація та десеріалізація

**Завдання**: Створіть структуру `User` з полями `name`, `email` та `birthdate`. Виконайте серіалізацію в JSON і десеріалізацію назад у структуру.

```rust
use serde::{Serialize, Deserialize};
use serde_json;

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    email: String,
    birthdate: String,
}

fn main() {
    // Створення екземпляра структури User
    let user = User {
        name: "Іван".to_string(),
        email: "ivan@example.com".to_string(),
        birthdate: "2000-01-01".to_string(),
    };

    // Серіалізація в JSON
    let json = serde_json::to_string(&user).expect("Помилка серіалізації");
    println!("Серіалізований JSON: {}", json);

    // Десеріалізація з JSON
    let deserialized_user: User = serde_json::from_str(&json).expect("Помилка десеріалізації");
    println!("Десеріалізований користувач: {:?}", deserialized_user);
}
```

---

### 2. Десеріалізація з файлу

**Завдання**: Створіть файл `user.json` з JSON-даними та прочитайте його в структуру `User`.

**user.json**

```json
{
  "name": "Марія",
  "email": "maria@example.com",
  "birthdate": "1995-05-15"
}
```

**Код:**

```rust
use std::fs;
use serde_json;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct User {
    name: String,
    email: String,
    birthdate: String,
}

fn main() {
    let data = fs::read_to_string("user.json").expect("Помилка читання файлу");
    let user: User = serde_json::from_str(&data).expect("Помилка десеріалізації");
    println!("Десеріалізований користувач: {:?}", user);
}
```

---

### 3. Підтримка різних форматів (JSON, YAML, TOML)

**Завдання**: Виконайте серіалізацію структури `User` у формат YAML і конвертуйте її назад у JSON.

```rust
use serde_yaml;

fn main() {
    let user = User {
        name: "Олексій".to_string(),
        email: "olexiy@example.com".to_string(),
        birthdate: "1987-12-31".to_string(),
    };

    // Серіалізація в YAML
    let yaml = serde_yaml::to_string(&user).expect("Помилка серіалізації у YAML");
    println!("Серіалізований YAML:\n{}", yaml);

    // Десеріалізація з YAML і серіалізація назад у JSON
    let deserialized_user: User = serde_yaml::from_str(&yaml).expect("Помилка десеріалізації з YAML");
    let json = serde_json::to_string(&deserialized_user).expect("Помилка серіалізації у JSON");
    println!("Конвертований у JSON:\n{}", json);
}
```

---

### 4. Використання деривацій та атрибутів

**Завдання**: Налаштуйте кастомні атрибути `rename` та `skip` для полів структури.

```rust
#[derive(Serialize, Deserialize, Debug)]
struct User {
    #[serde(rename = "user_name")]
    name: String,
    email: String,

    #[serde(skip)]
    password: String,  // Пропуститься при серіалізації
}

fn main() {
    let user = User {
        name: "Наталя".to_string(),
        email: "nataliia@example.com".to_string(),
        password: "секрет".to_string(),
    };

    let json = serde_json::to_string(&user).expect("Помилка серіалізації");
    println!("Серіалізований JSON без паролю: {}", json);
}
```

---

### 5. Кастомна серіалізація та десеріалізація

**Завдання**: Використовуйте власну логіку серіалізації для формату дати.

```rust
use serde::{Serializer, Deserializer};
use serde::de::Error as DeError;

#[derive(Debug)]
struct Event {
    name: String,
    #[serde(serialize_with = "serialize_date", deserialize_with = "deserialize_date")]
    date: String,
}

// Функція серіалізації дати
fn serialize_date<S>(date: &str, serializer: S) -> Result<S::Ok, S::Error>
where
    S: Serializer,
{
    serializer.serialize_str(&format!("Date: {}", date))
}

// Функція десеріалізації дати
fn deserialize_date<'de, D>(deserializer: D) -> Result<String, D::Error>
where
    D: Deserializer<'de>,
{
    let s: &str = Deserialize::deserialize(deserializer)?;
    Ok(s.replace("Date: ", ""))
}

fn main() {
    let event = Event {
        name: "Концерт".to_string(),
        date: "2024-11-15".to_string(),
    };

    let json = serde_json::to_string(&event).expect("Помилка серіалізації");
    println!("Серіалізований JSON з кастомною датою: {}", json);

    let deserialized_event: Event = serde_json::from_str(&json).expect("Помилка десеріалізації");
    println!("Десеріалізована подія: {:?}", deserialized_event);
}
```

---

### 6. Обробка помилок

**Завдання**: Обробіть помилки десеріалізації з кастомним повідомленням.

```rust
fn main() {
    let invalid_json = r#"{ "name": "Олена", "email": 12345 }"#;

    let user: Result<User, _> = serde_json::from_str(invalid_json);
    match user {
        Ok(user) => println!("Десеріалізований користувач: {:?}", user),
        Err(e) => println!("Помилка десеріалізації: {}", e),
    }
}
```
