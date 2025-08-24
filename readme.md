#  [🚀PostgreSQL Template Code 🚀](https://github.com/Saquib-Anjum/important-webdev-template-code/blob/main/postgresSQL_template_code.md)
#  [Git Command for security ⚒️](https://github.com/Saquib-Anjum/important-webdev-template-code/blob/main/securityGit.md)
# 📁 Git Ignore Template

This template includes common patterns to ignore files and directories in a Git project.

```gitignore
# Ignore node_modules directory
node_modules/

# Ignore environment variables and sensitive files
.env
.env.local
.env.*.local

# Ignore build outputs
dist/
build/

# Ignore logs
logs/
*.log
npm-debug.log*

# Ignore system files
.DS_Store
Thumbs.db

# Ignore IDE configurations
.vscode/
.idea/
*.iml

# Ignore temporary files
*.tmp
*.swp
*.bak

# Ignore coverage reports
coverage/

# Ignore custom debug files
debug.log
```

# printing server location
```
console.log(`Server is  running on address http://localhost:${PORT}`)
```
---
# DB Connection🍃🌿🍀🌲
```
DB_URL=mongodb://127.0.0.1:27017/<Database name >?authSource=admin&w=1
```

-way to connectDB
```javascript
// Connect to MongoDB
mongoose.connect(DB_URL).then(()=>{console.log('DB CONNECTED ')}).catch((err)=>{
console.log(err)
});
```
```javascript
import mongoose from "mongoose";
import dotenv from "dotenv";

dotenv.config();

const connectDB = async () => {
  try {
    mongoose.connection.on("connected", () => {
      console.log(" 💖 DB Connected");
    });
    await mongoose.connect(`${process.env.MONGODB_URI}`);
  } catch (err) {
    console.error(" Initial connection error:", err);
  }
};

export default connectDB;

});
```
---
## All the **NPM** Package for backend Project 🥷
```bash
npm install express cors dotenv mongoose pg pg-hstore sequelize jsonwebtoken bcryptjs express-validator cookie-parser multer helmet morgan compression uuid nodemailer winston

```
---
---
## 🚀 Vercel Deployment  Configuration  for express backend 🩺

This project is configured for deployment using **Vercel**. Below is the `vercel.json` configuration file used for deployment:

```json
  {
    "version": 2,
    "builds": [
        {
            "src": "server.js",
            "use": "@vercel/node",
            "config": {
                "includeFiles": [
                    "dist/**"
                ]
            }
        }
    ],
    "routes": [
        {
            "src": "/(.*)",
            "dest": "server.js"
        }
    ]
}
```
##🍀Vercel config for FrontEnd (react-router-dom)
```json

  {
    "rewrites": [
      {
        "source": "/(.*)",
        "destination": "/"
      }
    ]
  }
```
---
##  Cloudinary ☁️⛈️ Configuration in MERN Stack
```javascript
import { v2 as cloudinary } from "cloudinary";
const connectCloudinary = async () => {
  cloudinary.config({
    cloud_name: process.env.CLOUDINARY_NAME,
    api_key: process.env.CLOUDINARY_API_KEY,
    api_secret: process.env.CLOUDINARY_SECRET_KEY,
  });
};
export default connectCloudinary;

```
---
```javascript
import multer from "multer";
const storage = multer.diskStorage({
  filename: function (req, file, callback) {
    callback(null, file.originalname);
  },
});

const upload = multer({ storage });
export default upload;
```
