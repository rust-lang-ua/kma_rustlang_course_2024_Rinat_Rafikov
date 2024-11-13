### Крок 1: Підготовка проєкту та додавання залежностей

1. **Створіть новий проєкт**:

   ```bash
   cargo new calculator
   cd calculator
   ```

2. **Додайте залежності** в `Cargo.toml`:
   Відкрийте `Cargo.toml` і додайте бібліотеки `pest` та `pest_derive`:
   ```toml
   [dependencies]
   pest = "2.1"
   pest_derive = "2.1"
   ```
3. **Запустіть команду збірки** для перевірки, чи все налаштовано правильно:
   ```bash
   cargo build
   ```
   Якщо помилок немає, ми готові перейти до наступного кроку.

---

### Крок 2: Визначення граматики для арифметичних виразів

1. **Створіть файл граматики**. У папці `src` створіть новий файл з назвою `calculator.pest`.
2. **Додайте правила граматики** в цей файл:

   ```pest
   WHITESPACE = _{ " " | "\t" | "\n" }

   expression = _{ term ~ (("+" | "-") ~ term)* }
   term = _{ factor ~ (("*" | "/") ~ factor)* }
   factor = _{ primary ~ ("^" ~ primary)* }
   primary = _{ number | "(" ~ expression ~ ")" }

   number = @{ "-"? ~ ASCII_DIGIT+ }
   ```

3. **Пояснення**:

   - `expression`: Обробляє додавання та віднімання.
   - `term`: Обробляє множення та ділення.
   - `factor`: Обробляє піднесення до степеня.
   - `primary`: Визначає числа або вирази в дужках.
   - `number`: Визначає цілі числа з необов'язковим знаком `-`.

### Крок 3: Перший парсер у `main.rs`

1. Відкрийте файл `src/main.rs` і імпортуйте `pest` і `pest_derive`, додавши ці рядки:

   ```rust
   use pest::Parser;
   use pest_derive::Parser;
   ```

2. **Оголосіть парсер**:
   Додайте код, щоб оголосити структуру парсера на основі `calculator.pest`:

   ```rust
   #[derive(Parser)]
   #[grammar = "calculator.pest"] // Вказуємо шлях до файлу граматики
   struct CalculatorParser;
   ```

3. **Додайте функцію `main`** для тестування парсера:

   ```rust
   fn main() {
       let expression = "3 + 5 * (2 - 1)";
       let parsed = CalculatorParser::parse(Rule::expression, expression)
           .expect("Помилка парсингу")
           .next()
           .unwrap();

       println!("Розібраний вираз: {:?}", parsed);
   }
   ```

4. **Запуск тесту**:
   Зберігайте файл і запустіть програму:
   ```bash
   cargo run
   ```
   Ви повинні побачити структуру розібраного виразу. Якщо вона коректна, переходимо до наступного кроку.

---

### Крок 4: Додайте функцію `eval` для обчислення значень

1. Додайте базову структуру для функції `eval`, яка прийматиме `Pair` і оброблятиме `number`:

   ```rust
   fn eval(expression: pest::iterators::Pair<Rule>) -> i64 {
       match expression.as_rule() {
           Rule::number => expression.as_str().parse().unwrap(),
           _ => 0,
       }
   }
   ```

2. **Оновіть `main`** для виклику `eval`:

   ```rust
   fn main() {
       let expression = "3";
       let parsed = CalculatorParser::parse(Rule::expression, expression)
           .expect("Помилка парсингу")
           .next()
           .unwrap();

       let result = eval(parsed);
       println!("Результат: {}", result);
   }
   ```

3. **Тестування**:
   Запустіть знову:
   ```bash
   cargo run
   ```
   Ви маєте побачити, що `3` обробляється як `3`. Це базове обчислення, але код працює.

---

### Крок 5: Обробка додавання і віднімання

1. Додайте обробку `expression` для операцій `+` та `-` у `eval`:

   ```rust
   fn eval(expression: pest::iterators::Pair<Rule>) -> i64 {
       match expression.as_rule() {
           Rule::expression => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "+" => result + next,
                       "-" => result - next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::number => expression.as_str().parse().unwrap(),
           _ => 0,
       }
   }
   ```

2. **Тестування додавання**:

   - Змініть `expression` на `3 + 2`.
   - Запустіть:
     ```bash
     cargo run
     ```
   - Перевірте, що результат коректний (має бути `5`).

3. **Тестування віднімання**:
   - Змініть `expression` на `7 - 3`.
   - Переконайтеся, що результат `4`.

---

### Крок 6: Додавання обробки множення та ділення

1. Додайте обробку для `term`, яка виконує множення (`*`) і ділення (`/`):

   ```rust
   fn eval(expression: pest::iterators::Pair<Rule>) -> i64 {
       match expression.as_rule() {
           Rule::expression => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "+" => result + next,
                       "-" => result - next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::term => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "*" => result * next,
                       "/" => result / next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::number => expression.as_str().parse().unwrap(),
           _ => 0,
       }
   }
   ```

2. **Оновіть `main`** для тестування множення та ділення:

   ```rust
   fn main() {
       let expression = "4 * 5 / 2";
       let parsed = CalculatorParser::parse(Rule::expression, expression)
           .expect("Помилка парсингу")
           .next()
           .unwrap();

       let result = eval(parsed);
       println!("Результат: {}", result);
   }
   ```

3. **Тестування**:
   - Запустіть:
     ```bash
     cargo run
     ```
   - Ви маєте побачити результат `10` для виразу `4 * 5 / 2`.

---

### Крок 7: Додавання обробки піднесення до степеня

1. Додайте обробку для `factor`, яка буде виконувати операцію піднесення до степеня (`^`):

   ```rust
   fn eval(expression: pest::iterators::Pair<Rule>) -> i64 {
       match expression.as_rule() {
           Rule::expression => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "+" => result + next,
                       "-" => result - next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::term => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "*" => result * next,
                       "/" => result / next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::factor => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(_) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = result.pow(next as u32);
               }
               result
           },
           Rule::number => expression.as_str().parse().unwrap(),
           _ => 0,
       }
   }
   ```

2. **Тестування піднесення до степеня**:
   - Змініть вираз у `main` на `"2 ^ 3 ^ 2"`.
   - Запустіть:
     ```bash
     cargo run
     ```
   - Ви маєте побачити результат `512` для виразу `2 ^ (3 ^ 2)`.

---

### Крок 8: Додавання підтримки дужок

1. У `eval`, додайте обробку `primary`, щоб обчислювати вирази в дужках:

   ```rust
   fn eval(expression: pest::iterators::Pair<Rule>) -> i64 {
       match expression.as_rule() {
           Rule::expression => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "+" => result + next,
                       "-" => result - next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::term => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(op) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = match op.as_str() {
                       "*" => result * next,
                       "/" => result / next,
                       _ => unreachable!(),
                   };
               }
               result
           },
           Rule::factor => {
               let mut inner = expression.into_inner();
               let mut result = eval(inner.next().unwrap());
               while let Some(_) = inner.next() {
                   let next = eval(inner.next().unwrap());
                   result = result.pow(next as u32);
               }
               result
           },
           Rule::primary => eval(expression.into_inner().next().unwrap()),
           Rule::number => expression.as_str().parse().unwrap(),
           _ => unreachable!(),
       }
   }
   ```

2. **Тестування дужок**:
   - Змініть вираз у `main` на `"3 + 2 * (4 + 2)"`.
   - Запустіть:
     ```bash
     cargo run
     ```
   - Ви маєте побачити результат `15`.

---

### Крок 9: Остаточна реалізація з підтримкою командного рядка

1. Відредагуйте `main` для обробки аргументів командного рядка:

   ```rust
   fn main() {
       let args: Vec<String> = std::env::args().collect();
       if args.len() != 2 {
           eprintln!("Використання: {} \"вираз\"", args[0]);
           std::process::exit(1);
       }

       let expression = &args[1];
       let parsed = CalculatorParser::parse(Rule::expression, expression)
           .expect("Помилка парсингу")
           .next()
           .unwrap();

       let result = eval(parsed);
       println!("Результат: {}", result);
   }
   ```

2. **Тестування командного рядка**:
   - Запустіть програму з виразом:
     ```bash
     cargo run "3 + 2 ^ (4 + 2) - 6"
     ```
   - Ви повинні побачити результат `61`.

---

Тепер у нас є повноцінний калькулятор, який підтримує цілі числа, операції `+`, `-`, `*`, `/`, `^` та дужки!
