# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Subscriber model struct.`
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Subscriber repository.`
    -   [ ] Commit: `Implement list_all function in Subscriber repository.`
    -   [ ] Commit: `Implement delete function in Subscriber repository.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
1. In this BambangShop case, a single `Subscriber` model struct is enough for the current stage. Every subscriber stores the same data (`url` and `name`) and follows the same behavior, which is receiving notifications through an HTTP POST request. Because there is only one concrete subscriber behavior right now, adding a trait would introduce extra abstraction without much benefit. A trait would become useful later if we supported multiple observer types with different update mechanisms, such as webhooks, email notifications, or message-queue consumers.

2. Since `id` in `Product` and `url` in `Subscriber` are intended to be unique keys, using only `Vec` would work but would not be a good fit. With `Vec`, we would need to scan the list to check duplicates, search entries, and delete entries, so the uniqueness rule would be enforced manually and repeatedly. A map structure models this requirement better because the key itself represents uniqueness. In this project, `DashMap` makes lookup and deletion by key more direct, and for subscribers it fits the nested structure of `product_type -> url -> Subscriber`.

3. `DashMap` and Singleton solve different problems, so Singleton cannot replace `DashMap`. Singleton only guarantees that there is one shared instance of a data store, while `DashMap` guarantees that the shared data can be accessed and mutated safely from multiple threads. In fact, `lazy_static!` already gives us singleton-like access to `SUBSCRIBERS`, but we still need a thread-safe container inside that singleton because Rocket can handle multiple requests concurrently. If we did not use `DashMap`, we would still need another synchronization mechanism such as `Mutex` or `RwLock`. Therefore, in this case Singleton alone is not enough.

#### Reflection Publisher-2
1. We need to separate `Service` and `Repository` from the Model because they represent different responsibilities. The model should focus on domain data and simple domain-related behavior, the repository should focus on data access and storage, and the service should coordinate business rules and application use cases. In this project, for example, the controller handles HTTP requests, `NotificationService` handles subscription logic such as normalizing `product_type` and deciding what to return, and `SubscriberRepository` manages how subscribers are stored. This separation follows single responsibility and separation of concerns, and it makes the code easier to test, maintain, and extend.

2. If we only used the Model, each model would become responsible for too many things at once. `Product` would not only represent product data, but also handle storage operations and trigger notification-related workflows. `Subscriber` would need to store itself, remove itself, and possibly know too much about how subscriptions are grouped. `Notification` would stop being a simple payload model and become a place for orchestration logic. As the interaction between these models grows, the code would become more tightly coupled, harder to reason about, and harder to modify because a small change in one feature could affect multiple unrelated responsibilities inside the same model.

3. Postman helps a lot in this tutorial because it makes endpoint testing fast and repeatable. I can send requests to `subscribe` and `unsubscribe`, verify the returned status code and JSON body, and quickly try both valid and invalid cases without writing a separate client application. The most useful features for me are collections, saved request templates, environment variables, and request history. For future group projects, I am also interested in Postman tests and collection runners because they can help turn manual API checks into repeatable regression tests.

#### Reflection Publisher-3
