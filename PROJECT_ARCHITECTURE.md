# YelpCamp Architecture & Data Flow Guide

Welcome to the comprehensive architecture and design reference for **YelpCamp**. This document details the application's overall system design, technology stack, directory organization, database models, security features, and interactive data flow processes.

---

## 1. System Overview

YelpCamp is a full-stack, server-side rendered (SSR) web application where users can share, review, and view campgrounds around the world. It is built following the traditional **Model-View-Controller (MVC)** architectural pattern using **Node.js** and **Express**.

The system integrates with cloud databases, object storage providers, maps, and geocoding services:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '16px', 'fontFamily': 'Arial, sans-serif', 'primaryColor': '#ffffff', 'primaryTextColor': '#111111', 'primaryBorderColor': '#333333', 'lineColor': '#333333', 'tertiaryColor': '#ffffff', 'mainBkg': '#ffffff', 'nodeBorder': '#333333'}}}%%
flowchart TD
    Client["Browser / Client"] <--> Express["Express.js Web Server"]
    Express <--> MongoDB[("MongoDB database")]
    Express <--> Cloudinary["Cloudinary CDN"]
    Express <--> Mapbox["Mapbox Geocoding API"]
```

---

## 2. Technology Stack

The application's technical stack is organized into distinct functional layers:

| Layer | Component / Tool | Technology / Package | Purpose |
| :--- | :--- | :--- | :--- |
| **Server Engine** | Web Framework | `express` | Handles Routing, HTTP Requests, Middleware, and Controllers |
| **Database** | Database Engine | `MongoDB` | Document-oriented database for flexible schema models |
| | Object Data Modeler (ODM) | `mongoose` | Schema definitions, validation, hooks, and queries |
| | Session Storage | `connect-mongo` | Persists session cookies in MongoDB instead of in-memory |
| **View Template** | View Engine | `ejs` & `ejs-mate` | Embedded Javascript templates with page inheritance layouts |
| | Maps | `Mapbox GL JS (v1.12.0)` | Renders interactive maps on the client side |
| **Authentication** | Authentication Engine | `passport` | Middleware to handle login sessions and requests |
| | Local Strategy | `passport-local` | Authenticates users against username/password in database |
| | Mongoose Plugin | `passport-local-mongoose` | Automates hashing, salting, and security fields on User model |
| **Cloud Services** | Image CDN | `cloudinary` | Stores user-uploaded campgrounds images |
| | Map Service | `@mapbox/mapbox-sdk` | Forward geocoding of location strings (e.g. "Paris" to coordinates) |
| | Multi-part Uploads | `multer` & `multer-storage-cloudinary` | Parses file forms and pipes files directly to Cloudinary |
| **Security & Validation** | Content Security Policy | `helmet` | Sets HTTP response headers to prevent XSS, clickjacking |
| | Input Validation | `joi` | Validates req.body schema limits on server side |
| | HTML Sanitization | `sanitize-html` | Trims out script tags and HTML injection vectors from text |
| | Injection Prevention | `express-mongo-sanitize` | Strips out characters starting with `$` or `.` from query objects |

---

## 3. Architecture & Code Structure (MVC)

YelpCamp strictly implements the **Model-View-Controller (MVC)** pattern to separate concern layers:

- **Routing Layer (`/routes`)**: Intercepts HTTP requests, validates permissions (e.g. authentication, author authorization), applies request body validations via Joi schemas, and maps URLs to controller logic.
- **Controller Layer (`/controllers`)**: Performs the core business workflows, queries database schemas via Mongoose, interfaces with external APIs (Mapbox and Cloudinary), populates views with data payload, and triggers SSR view rendering.
- **Model Layer (`/models`)**: Defines schema properties, specifies database validation parameters, embeds virtual fields, and handles lifecycle hooks (like cascade deleting campground reviews).
- **View Layer (`/views`)**: Server-side templates containing HTML structured with Embedded JavaScript (EJS). Extends standard layouts (boilerplate) and includes reusable partials (navbars, flash alerts).

### Directory Breakdown
- [app.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/app.js): Main setup, configuration, database connection, middleware orchestration
- [middleware.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/middleware.js): Global permission-checking and authorization middlewares
- [schemas.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/schemas.js): Joi schemas for server-side body validation and sanitation
- [cloudinary/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/cloudinary/): Cloudinary configuration and storage connection configuration
- [controllers/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/controllers/): MVC Controllers containing backend business workflows
- [models/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/models/): Mongoose DB schema definitions
- [routes/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/routes/): Express route parameters and method routes
- [seeds/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/seeds/): Local seed datasets and script to initialize database state
- [utils/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/utils/): Error wrappers and async exception handlers
- [public/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/public/): Static frontend scripts, styles, and image resources
- [views/](file:///c:/Users/KIIT0001/Desktop/YelpCamp/views/): EJS pages, layouts, and partial templates

---

## 4. Database Architecture & Relationships

YelpCamp uses MongoDB as its primary persistence store. Below is the Entity-Relationship (ER) model and schema specification:

### Entity-Relationship Diagram

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '16px', 'fontFamily': 'Arial, sans-serif', 'primaryColor': '#ffffff', 'primaryTextColor': '#111111', 'primaryBorderColor': '#333333', 'lineColor': '#333333', 'entityBackground': '#ffffff', 'attributeBackgroundColorOdd': '#ffffff', 'attributeBackgroundColorEven': '#ffffff'}}}%%
erDiagram
    direction LR
    USER {
        ObjectId id PK
        string email
        string username
        string hash
        string salt
    }
    CAMPGROUND {
        ObjectId id PK
        string title
        string description
        string location
        number price
        object geometry
        array images
        ObjectId author FK
    }
    REVIEW {
        ObjectId id PK
        string body
        number rating
        ObjectId author FK
    }

    USER ||--o{ CAMPGROUND : "creates"
    USER ||--o{ REVIEW : "writes"
    CAMPGROUND ||--o{ REVIEW : "has"
```

---

### Detailed Schema Specifications

#### A. Campground Model ([campground.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/models/campground.js))

Tracks details of user-submitted campgrounds, including geocoded location, image links, owner, and review collections.

| Field Name | Data Type | Relationships / Description | Notes |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Primary Key | Auto-generated by MongoDB |
| `title` | `String` | Required | validated on submit |
| `description` | `String` | Required | validated on submit |
| `location` | `String` | Textual description of campground location | E.g. "Yosemite Valley, CA" |
| `price` | `Number` | Price per night | Minimum 0 validation |
| `geometry` | `Object` | GeoJSON representation of coordinates | Mapped to `Point` coordinates: `[longitude, latitude]` for maps |
| `images` | `Array` | List of `ImageSchema` | Images upload to Cloudinary. Each has a URL and a Cloudinary filename |
| `author` | `ObjectId` | Reference $\rightarrow$ `User` | Links to campground creator |
| `reviews` | `Array` | References $\rightarrow$ `[Review]` | Array of Review ObjectIds linked to this campground |

* **Virtuals**:
  - `images[].thumbnail`: Generates a modified Cloudinary URL specifying width parameters (`w_200`) dynamically on query.
  - `properties.popUpMarkup`: Constructs a clickable raw HTML tag (`<a href="/campgrounds/ID">Title</a>`) embedded in campground data for Mapbox interactive cluster map popup window.
* **Mongoose Hooks**:
  - `post('findOneAndDelete')`: Automatically triggers when a Campground is deleted. Performs a cascade delete, purging all linked reviews from the reviews collection in the database.

---

#### B. Review Model ([review.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/models/review.js))

Represents a review added to a campground.

| Field Name | Data Type | Relationships / Description | Notes |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Primary Key | Auto-generated |
| `body` | `String` | Content of the review | Sanitized to prevent HTML injections |
| `rating` | `Number` | Score from 1 to 5 | Required |
| `author` | `ObjectId` | Reference $\rightarrow$ `User` | Owner of this specific review |

---

#### C. User Model ([user.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/models/user.js))

Represents user accounts, authenticated via `passport-local-mongoose`.

| Field Name | Data Type | Relationships / Description | Notes |
| :--- | :--- | :--- | :--- |
| `_id` | `ObjectId` | Primary Key | Auto-generated |
| `email` | `String` | Unique, Required | Used for registration and credentials verification |
| `username` | `String` | Unique | Added automatically by Passport plugin |
| `hash` | `String` | Hashed password digest | Added automatically by Passport plugin |
| `salt` | `String` | Security salt buffer | Added automatically by Passport plugin |

---

## 5. Security & Request Validation

YelpCamp implements multi-tier security boundaries on incoming data, session persistence, and headers:

1. **Strict Input Sanitation & Schema Validation ([schemas.js](file:///c:/Users/KIIT0001/Desktop/YelpCamp/schemas.js))**:
   - Integrated `Joi` validation schemas on all campgrounds and reviews creations/updates.
   - Built custom `Joi` extension running `sanitize-html` to block and strip out HTML injections (`<script>` elements) from any string body parameters.
2. **MongoDB Injection Protection (`express-mongo-sanitize`)**:
   - Intercepts requests and replaces any user queries containing illegal characters (`$` or `.`) with an underscore `_` to block MongoDB injection exploits.
3. **Secure Persistent Sessions (`connect-mongo` + `express-session`)**:
   - Instead of storing server-session variables in-memory (which creates memory leaks), it serializes them directly to a session database collection in MongoDB using `connect-mongo`.
   - Cookie setup includes `httpOnly: true` (restricts Javascript from accessing cookies to prevent XSS session theft) and configures automatic cookie expiry.
4. **Header and CSP Security (`helmet`)**:
   - Initializes 15 middlewares under `helmet()` to set standard headers (strict-transport-security, dns-prefetch-control, x-content-type, etc.).
   - Configures a restrictive **Content Security Policy (CSP)** specifying strict white-listed source domains (such as `api.mapbox.com` and `res.cloudinary.com`) to block cross-site execution vectors.

---

## 6. End-to-End Data Flow Diagrams

The following charts outline the request and processing flows inside the application.

### A. User Authentication Flow
Demonstrates registration, hashing verification, and user session storage.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '16px', 'fontFamily': 'Arial, sans-serif', 'actorBkg': '#ffffff', 'actorBorder': '#333333', 'actorTextColor': '#111111', 'actorLineColor': '#333333', 'signalColor': '#333333', 'signalTextColor': '#111111', 'labelBoxBkgColor': '#ffffff', 'labelBoxBorderColor': '#333333', 'labelTextColor': '#111111', 'loopTextColor': '#111111', 'noteBkgColor': '#ffffff', 'noteBorderColor': '#333333', 'noteTextColor': '#111111', 'activationBkgColor': '#ffffff', 'activationBorderColor': '#333333', 'sequenceNumberColor': '#333333'}}}%%
sequenceDiagram
    autonumber
    actor User as Client / User
    participant Router as Express Router
    participant Passport as Passport.js
    participant Model as User Model
    participant DB as MongoDB

    User->>Router: POST /register (email, username, password)
    Router->>Model: Create User instance with Email & Username
    Model->>Passport: User.register(user, password)
    Note over Passport: Automatically generates salt<br/>and hashes the password
    Passport->>DB: Save User documents (including hash & salt)
    DB-->>Passport: Confirms Save Success
    Passport-->>Router: Returns Registered User Profile
    Router->>User: Auto-login user (serializes user to Session Cookie) & Redirect
```

---

### B. Campground Creation Flow
Illustrates form parsing, multi-destination upload (Cloudinary and Mapbox), database persistence, and views updates.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '16px', 'fontFamily': 'Arial, sans-serif', 'actorBkg': '#ffffff', 'actorBorder': '#333333', 'actorTextColor': '#111111', 'actorLineColor': '#333333', 'signalColor': '#333333', 'signalTextColor': '#111111', 'labelBoxBkgColor': '#ffffff', 'labelBoxBorderColor': '#333333', 'labelTextColor': '#111111', 'loopTextColor': '#111111', 'noteBkgColor': '#ffffff', 'noteBorderColor': '#333333', 'noteTextColor': '#111111', 'activationBkgColor': '#ffffff', 'activationBorderColor': '#333333', 'sequenceNumberColor': '#333333', 'altSectionBkgColor': '#ffffff'}}}%%
sequenceDiagram
    autonumber
    actor User as Client / User
    participant Router as Express Router
    participant Mid as middleware.isLoggedIn
    participant Multer as Multer (Storage Engine)
    participant Cloud as Cloudinary
    participant Maps as Mapbox API
    participant Joi as Joi Validator (validateCampground)
    participant Controller as Campgrounds Controller
    participant DB as MongoDB
    
    User->>Router: POST /campgrounds (multi-part form-data: title, location, images, details)
    Router->>Mid: Check Authentication status
    alt Not Logged In
        Mid-->>User: Redirect to /login (with Flash warning)
    else Authenticated
        Mid->>Multer: Parse upload fields and files
        Multer->>Cloud: Upload image files
        Cloud-->>Multer: Return URLs & file IDs
        Multer->>Joi: Validate parsed fields (Joi.validate)
        alt Validation Fails
            Joi-->>Router: Throw ExpressError (400 Bad Request)
        else Validation Success
            Joi->>Controller: Invokes createCampground()
            Note over Controller: Extracts Location string
            Controller->>Maps: forwardGeocode(location)
            Maps-->>Controller: Return Geometry JSON (Coordinates)
            Note over Controller: Maps Cloudinary links & Geometry<br/>into Campground Instance
            Controller->>DB: Save campground
            DB-->>Controller: Success
            Controller-->>User: Set Success flash & Redirect to /campgrounds/:id
        end
    end
```

---

### C. Review Creation Flow
Highlights review submission, association with campgrounds, and data population.

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '16px', 'fontFamily': 'Arial, sans-serif', 'actorBkg': '#ffffff', 'actorBorder': '#333333', 'actorTextColor': '#111111', 'actorLineColor': '#333333', 'signalColor': '#333333', 'signalTextColor': '#111111', 'labelBoxBkgColor': '#ffffff', 'labelBoxBorderColor': '#333333', 'labelTextColor': '#111111', 'loopTextColor': '#111111', 'noteBkgColor': '#ffffff', 'noteBorderColor': '#333333', 'noteTextColor': '#111111', 'activationBkgColor': '#ffffff', 'activationBorderColor': '#333333', 'sequenceNumberColor': '#333333', 'altSectionBkgColor': '#ffffff'}}}%%
sequenceDiagram
    autonumber
    actor User as Client / User
    participant Router as Express Router
    participant Mid as middleware.isLoggedIn
    participant Joi as Joi Validator (validateReview)
    participant Controller as Reviews Controller
    participant DB as MongoDB

    User->>Router: POST /campgrounds/:id/reviews (body, rating)
    Router->>Mid: Check authentication status
    Mid->>Joi: Validate review rating and body constraints
    alt Validation fails
        Joi-->>Router: Throw Joi error (400)
    else Validation passes
        Joi->>Controller: createReview()
        Controller->>DB: Find Campground by ID
        DB-->>Controller: Return Campground Instance
        Note over Controller: Create Review instance & link author = req.user._id
        Controller->>DB: Save Review Document
        Controller->>DB: Push Review ObjectId to Campground.reviews & Save
        DB-->>Controller: Save Confirmed
        Controller-->>User: Set Flash success & Redirect to show page
    end
```

---

## 7. Dynamic UI & Maps Binding Flow

To render client-side maps with live server-side database records, YelpCamp maps Mongoose queries to frontend JavaScript scripts:

```mermaid
%%{init: {'theme': 'base', 'themeVariables': {'fontSize': '16px', 'fontFamily': 'Arial, sans-serif', 'primaryColor': '#ffffff', 'primaryTextColor': '#111111', 'primaryBorderColor': '#333333', 'lineColor': '#333333', 'edgeLabelBackground': '#ffffff', 'tertiaryColor': '#ffffff', 'mainBkg': '#ffffff', 'nodeBorder': '#333333'}}}%%
flowchart TD
    A["User requests /campgrounds"] --> B("Controller loads all campgrounds from DB")
    B --> C("Controller returns campgrounds collection to EJS index view")
    C --> D["EJS compiles variable: const campgrounds = { features: JSON.stringify(campgrounds) }"]
    D --> E["EJS imports public/javascripts/clusterMap.js"]
    E --> F["clusterMap.js reads campgrounds as a GeoJSON FeatureCollection"]
    F --> G["Mapbox GL JS draws cluster circles and unclustered map pins on client screen"]
```
