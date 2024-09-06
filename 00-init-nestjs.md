# Setup NestJS, CLI, GitHub-link
[NestJS install](https://docs.nestjs.com/#installation)
```bash
npm i -g @nestjs/cli@latest
nest --version
```

# Create repo on GitHub (nestjs-books-backend for example)

```bash
cd nestjs-books-backend/
echo "# nestjs-books-backend" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/your-name/nestjs-books-backend.git
git push -u origin main
```
Delete README.md

# Create branches: develop and work branch, add .gitignore


```bash
nest new nestjs-books-backend --directory ./
```

# Add .github/pull_request_template.md

```bash
1. Task: [link](https://github.com/)
2. Screenshot:
3. Deploy: [link](https://github.com/)
4. Done 00.09.2024 / deadline 0.09.2024
5. Score: 100 / 100
- [x] Application is 
```

# Add dependencies:

```bash
npm install @nestjs/jwt @nestjs/mapped-types @nestjs/passport @nestjs/typeorm @types/bcrypt bcrypt class-transformer
npm install class-validator passport passport-jwt passport-local pg reflect-metadata typeorm
npm install -D @types/passport-jwt @types/passport-local
```

# Create branch 01-start-project
## add .env file
```bash
DB_HOST=localhost
DB_PORT=5432
DB_NAME=bookstall   # Имя вашей базы данных
DB_USER=postgres    # Имя пользователя базы данных
DB_PASSWORD=some-password    # Пароль пользователя базы данных
DB_TYPE=postgres

PORT=5001 #порт приложения

```
