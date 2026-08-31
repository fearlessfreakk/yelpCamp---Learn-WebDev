# YelpCamp 🏕️

YelpCamp is a full-stack web application designed for outdoor enthusiasts to share, rate, and review campgrounds globally. Built with Node.js, Express, and MongoDB, it features interactive maps, secure user authentication, robust input validation, and cloud-based image management.

## 🚀 Live Demo & Repository
* **GitHub Repository**: [fearlessfreakk/yelpCamp---Learn-WebDev](https://github.com/fearlessfreakk/yelpCamp---Learn-WebDev.git)

---

## ✨ Features

- **Campground Management**: Create, edit, delete, and browse campgrounds. Include details like title, description, pricing, location, and upload multiple photos.
- **Review System**: Registered users can rate campgrounds (1-5 stars) and write text reviews. Review authors can delete their reviews.
- **Geocoding & Interactive Maps**: Integrated with the Mapbox API to automatically convert text-based locations into geographic coordinates. Features interactive cluster maps for discovery and single-pin maps on individual campground pages.
- **User Authentication & Profiles**: Secure registration and login flows powered by Passport.js.
- **Image Management**: Seamless multi-image upload directly to Cloudinary storage. Optimized thumbnails are loaded via Cloudinary transformation URLs.
- **Input Security & Sanitization**: Restricts raw HTML injections (XSS mitigation) using `sanitize-html` and Joi schemas. Sanitizes query parameters against MongoDB operator injections.
- **Persistent Sessions**: Sessions are safely stored in a MongoDB collection using `connect-mongo`.

---

## 🛠️ Technology Stack

| Layer | Technologies Used |
| :--- | :--- |
| **Backend** | Node.js, Express.js |
| **Database** | MongoDB, Mongoose ODM |
| **Frontend** | Embedded JavaScript (EJS) templates, Bootstrap 5, Vanilla CSS, JS |
| **APIs & Services** | Mapbox GL JS, Cloudinary Image Storage |
| **Security & Auth** | Passport.js, Helmet.js, Joi, Express-Mongo-Sanitize |

*For a detailed look at the application structure, data flow sequences, and system boundaries, refer to the [PROJECT_ARCHITECTURE.md](file:///c:/Users/KIIT0001/Desktop/YelpCamp/PROJECT_ARCHITECTURE.md) guide.*

---

## 📋 Prerequisites

Before running YelpCamp locally, ensure you have the following installed:
- [Node.js](https://nodejs.org/) (v14+ recommended)
- [MongoDB](https://www.mongodb.com/try/download/community) (running locally or via MongoDB Atlas)
- Accounts with **Cloudinary** and **Mapbox** for API access tokens.

---

## 🔧 Installation & Setup

Follow these steps to set up YelpCamp on your local machine:

1. **Clone the repository**:
   ```bash
   git clone https://github.com/fearlessfreakk/yelpCamp---Learn-WebDev.git
   cd yelpCamp
   ```

2. **Install project dependencies**:
   ```bash
   npm install
   ```

3. **Configure Environment Variables**:
   Create a `.env` file in the root directory and define the following variables:
   ```env
   # Database url (default local fallback is used if omitted)
   DB_URL=mongodb://localhost:27017/yelp-camp
   SECRET=yoursupersecuresecretkey
   
   # Cloudinary Credentials
   CLOUDINARY_CLOUD_NAME=your_cloudinary_name
   CLOUDINARY_KEY=your_cloudinary_key
   CLOUDINARY_SECRET=your_cloudinary_secret
   
   # Mapbox Token
   MAPBOX_TOKEN=your_mapbox_public_token
   ```
   > ⚠️ **Important**: Make sure your `.env` file is added to your `.gitignore` to avoid exposing credentials.

4. **Seed the database (Optional)**:
   Initialize the database with 300 mock campgrounds by running the seeding script:
   ```bash
   node seeds/index.js
   ```

5. **Start the server**:
   ```bash
   npm start
   ```

6. **Open in browser**:
   Navigate to [http://localhost:3000](http://localhost:3000) to view the application.

---

## 🔐 Security Measures

YelpCamp implements industry-standard safety practices:
- **XSS Protections**: Restricts cross-site script injections inside custom Joi schemas.
- **SQL/NoSQL Injections**: Strips characters starting with `$` and `.` to keep queries safe.
- **Secure Sessions**: Persists cookies with `httpOnly: true` properties.
- **Content Security Policies**: Restricts browser requests to white-listed resource domains via Helmet CSP configuration.

---

## 📄 License

This project is licensed under the ISC License. Feel free to use and modify it for educational purposes.
