# 🎲 Shuffle Test Application

A high-performance, database-agnostic “shuffled pagination” demo built with **Spring Boot**.
This project demonstrates how to fetch deterministic random subsets of data using hashing (`MD5(key + id)`) in **multiple relational databases** while maintaining consistency across requests.

---

## 🚀 Features

* Deterministic shuffled pagination using a *key-based hash*
* Support for **PostgreSQL**, **MySQL/MariaDB**, **Oracle** and **SQL Server**
* REST endpoint to:

  * generate the next key
  * fetch shuffled items
  * seed the database with 10,000 records
* Stress test ensuring **no duplicates across 1000 iterations**
* Fully implemented repository queries per database
* Simple JSON responses with pagination metadata

---

## 📦 Tech Stack

* **Java 25**
* **Spring Boot 4+**
* **Spring Data JPA**
* **PostgreSQL (default)**
  *(other DBs supported via additional repository queries)*
* **Lombok**

---

## 📁 Project Structure

```
src/main/java/
  └── tr.kontas.shuffle_test/
        ├── Item.java
        ├── ItemRepository.java
        ├── ItemService.java
        ├── ItemController.java
        └── ShuffleTestApplication.java
```

---

## 🔑 How Shuffle Pagination Works

Every item has a fixed ID.

A “shuffle key” (string) is chosen.
Items are ordered by the hash:

```
ORDER BY md5(key + id)
```

This gives a **stable shuffle**:

* same `key` → same order
* different `key` → new shuffled order
* next key generated deterministically

This allows accessing pages randomly without expensive offsets.

---

## 📚 Database Queries

Each database requires a different hash syntax.

| Database   | Status | Function                           |
| ---------- | ------ | ---------------------------------- |
| PostgreSQL | ✅      | md5(:key || id)                    |
| MySQL      | ✅      | md5(CONCAT(:key, id))              |
| SQL Server | ✅      | HASHBYTES('MD5', CONCAT...)        |
| Oracle     | ✅      | DBMS_CRYPTO.HASH(...)              |

All query variants are implemented in the repository.

---

## 🧪 Seeding (10,000 items)

The controller exposes:

```
POST /items/seed
```

This fills the database with 10,000 items.

---

## 📄 Example Response

```
{
  "items": [ ... 10 shuffled items ... ],
  "key": "abc123",
  "nextKey": "cdef901234..."
}
```

---

## 🔥 Stress Test

A JUnit test runs **1000 iterations**, each time:

* generates a random key
* retrieves 10 shuffled items
* ensures:

  * same key → same result,
  * different key → no overlap collisions

This confirms correctness for real-world use cases.

---

## 🛠 Configuration (PostgreSQL default)

`src/main/resources/application.properties`

```
spring.datasource.url=jdbc:postgresql://localhost:5432/postgres
spring.datasource.username=postgres
spring.datasource.password=postgres
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

---

## ▶ Running the App

```bash
./mvnw spring-boot:run
```

Endpoints:

| Method | Path             | Description                         |
| ------ | ---------------- | ----------------------------------- |
| `POST` | `/items/seed`    | Inserts 10,000 items                |
| `GET`  | `/items?key=...` | Returns shuffled 10 items + nextKey |

---

## 📘 License

This project is provided for educational and experimental purposes.
