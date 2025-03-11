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
# DB Connection🍃🌿🍀🌲
```
DB_URL=mongodb://127.0.0.1:27017/<Database name >?authSource=admin&w=1
```
-way to connectDB
```
// Connect to MongoDB
mongoose.connect(DB_URL).then(()=>{console.log('DB CONNECTED ')}).catch((err)=>{
console.log(err)
});
```

## All the **NPM** Package for backend Project 🥷
```bash
npm install express cors dotenv mongoose pg pg-hstore sequelize jsonwebtoken bcryptjs express-validator cookie-parser multer helmet morgan compression uuid nodemailer winston

```
