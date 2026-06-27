## DevOps Jenkins & GitHub Actions & N8N - Day 7

### 📋 สารบัญ

1. [Jenkins multibranch pipeline](#jenkins-multibranch-pipeline)
2. [Workshop Jenkins multibranch pipeline](#workshop-jenkins-multibranch-pipeline)
3. [NodeJS Express Jenkins multibranch pipeline](#nodejs-express-jenkins-multibranch-pipeline)
4. [Python Flask Jenkins multibranch pipeline](#python-flask-jenkins-multibranch-pipeline)
5. [Java Spring Boot Jenkins multibranch pipeline](#java-spring-boot-jenkins-multibranch-pipeline)
6. [NextJS Jenkins multibranch pipeline](#nextjs-jenkins-multibranch-pipeline)
7. [.NET Core Jenkins multibranch pipeline](#dotnet-core-jenkins-multibranch-pipeline)
8. [Go Fiber Jenkins multibranch pipeline](#go-fiber-jenkins-multibranch-pipeline)
9. [PHP Laravel Jenkins multibranch pipeline](#php-laravel-jenkins-multibranch-pipeline)
10. [NestJS Jenkins multibranch pipeline](#nestjs-jenkins-multibranch-pipeline)

## Jenkins multibranch pipeline

> Jenkins Multibranch Pipeline คือฟีเจอร์ที่ช่วยให้เราสามารถสร้าง Pipeline ที่สามารถทำงานกับหลายๆ สาขา (branches) ของโค้ดในระบบควบคุมเวอร์ชัน เช่น Git ได้อย่างง่ายดาย

### ข้อดีของ Jenkins Multibranch Pipeline
1. **การจัดการหลายสาขาได้ง่าย**: สามารถสร้าง Pipeline สำหรับแต่ละสาขาได้โดยอัตโนมัติ
2. **การตรวจสอบคุณภาพโค้ด**: สามารถรันการทดสอบและการตรวจสอบคุณภาพโค้ดสำหรับแต่ละสาขาได้
3. **การปรับปรุงอย่างต่อเนื่อง**: สนับสนุนการพัฒนาแบบ Agile และ CI/CD ได้ดี
4. **การแยกสภาพแวดล้อม**: สามารถแยกสภาพแวดล้อมการพัฒนา การทดสอบ และการผลิตได้อย่างชัดเจน

### การตั้งค่า Jenkins Multibranch Pipeline
1. ติดตั้ง Jenkins และ Plugins ที่จำเป็น เช่น Git, GitHub, Pipeline, Pipeline Utility Steps, HTML Publisher
2. สร้าง Multibranch Pipeline Job ใหม่
3. ตั้งค่า Repository URL และ Credentials
4. กำหนด Branch Sources และ Strategies
5. บันทึกและรัน Pipeline

## Workshop Jenkins multibranch pipeline
1. สร้าง Repository ใหม่ใน GitHub สำหรับโค้ดตัวอย่าง
2. สร้าง Jenkins Multibranch Pipeline Job ใหม่
3. ตั้งค่า Repository URL และ Credentials
4. กำหนด Branch Sources และ Strategies
5. สร้าง Jenkinsfile สำหรับแต่ละสาขา
6. บันทึกและรัน Pipeline
7. ตรวจสอบผลลัพธ์และแก้ไขปัญหาที่เกิดขึ้น

## เตรียมโปรเจ็กต์ตัวอย่าง

```bash
git clone https://github.com/iamsamitdev/devops_workshops
```

### NodeJS Express Jenkins multibranch pipeline

### 🏗️ Project Structure

```
express-docker-app/
├── 📁 src/
│   └── 📄 app.ts                   # Main Express application (TypeScript)
├── 📁 tests/
│   └── 📄 app.test.ts              # Jest test suite with Supertest
├── 📁 dist/                        # Compiled JavaScript output
│   └── 📄 app.js                   # Compiled application
├── 📁 node_modules/                # Node.js dependencies
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 main.yml            # GitHub Actions workflow
├── 📄 .dockerignore                # Files to ignore in Docker build
├── 📄 .gitignore                   # Files to ignore in Git
├── 🐳 Dockerfile                   # Docker build configuration
├── 🐳 docker-compose.dev.yml       # Docker Compose for development
├── 🔧 Jenkinsfile                  # Jenkins CI/CD pipeline
├── ⚙️ jest.config.js              # Jest testing configuration
├── 📄 package.json                # Node.js project configuration
├── 📄 package-lock.json           # Dependency lock file
├── 📄 tsconfig.json               # TypeScript configuration
└── 📄 README.md                   # Project documentation
```

#### 1. Dockerfile

```dockerfile
# Build stage - สำหรับ development และ testing
FROM node:22-alpine AS builder

# กำหนด Working Directory ภายใน Container
WORKDIR /app

# Copy ไฟล์ package.json และ package-lock.json เข้าไปก่อน
# เพื่อใช้ประโยชน์จาก Docker cache layer ทำให้ไม่ต้อง install dependencies ใหม่ทุกครั้งที่แก้โค้ด
COPY package*.json ./

# ติดตั้ง Dependencies (รวม dev dependencies สำหรับ testing)
RUN npm install

# Copy โค้ดทั้งหมดในโปรเจกต์เข้าไปใน container
COPY . .

# Compile TypeScript เป็น JavaScript
RUN npm run build

# Production stage - สำหรับ production deployment
FROM node:22-alpine AS production

# กำหนด Working Directory ภายใน Container
WORKDIR /app

# Copy package files
COPY package*.json ./

# ติดตั้งเฉพาะ production dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy โค้ดที่ compiled แล้วจาก builder stage
COPY --from=builder /app/dist ./dist
# COPY --from=builder /app/src ./src

# กำหนด Port ที่ Container จะทำงาน
EXPOSE 3000

# คำสั่งสำหรับรัน Express Application (ใช้ compiled JavaScript)
CMD ["npm", "start"]
```

#### 2. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # สร้าง image จาก Dockerfile แต่จะใช้แค่ 'builder' stage เป็นฐาน
    # เพื่อให้มี devDependencies ครบ
    build:
      context: .
      target: builder # <-- บอกให้ build ถึงแค่ stage 'builder'
    container_name: ts-app-dev-instance
    ports:
      - "3002:3000"
    volumes:
      # เชื่อมโค้ดจากเครื่องเราเข้าไปใน container แบบ Real-time
      - ./src:/app/src
      - ./package.json:/app/package.json
      - ./package-lock.json:/app/package-lock.json
      - ./tsconfig.json:/app/tsconfig.json
      # ไม่ต้อง map node_modules เพื่อให้ใช้ของใน container
      - /app/node_modules
    # สั่งให้รันด้วย script "dev" ที่เราสร้างไว้
    command: npm run dev
```

#### 3. .dockerignore

```dockerignore
# Dependencies
node_modules
npm-debug.log*

# Build outputs
dist
build

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Testing
coverage
*.lcov

# Git
.git
.gitignore

# Docker
Dockerfile
.dockerignore

# Documentation
README.md
*.md

# IDE
.vscode
.idea
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
logs
*.log

# Temporary files
.tmp
.temp
```

#### 4. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: สร้างฟังก์ชันสำหรับส่ง Notification ไปยัง n8n
// การสร้างฟังก์ชันช่วยลดการเขียนโค้ดซ้ำซ้อน (DRY Principle)
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    // ใช้ Jenkins HTTP Request Plugin (ต้องติดตั้งก่อน)
    // หรือใช้ Java URLConnection แทน (fallback) ถ้า httpRequest ไม่ได้ติดตั้ง
    // n8n-webhook คือ Jenkins Secret Text Credential ที่เก็บ URL ของ n8n webhook
    // ต้องสร้าง Credential นี้ใน Jenkins ก่อน ใช้งาน
    // โดยใช้ ID ว่า n8n-webhook
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    // ใช้ agent any เพราะ build จะทำงานบน Jenkins controller (Linux container) อยู่แล้ว
    agent any

    // กัน “เช็คเอาต์ซ้ำซ้อน”
    // ถ้า job เป็นแบบ Pipeline from SCM / Multibranch แนะนำเพิ่ม options { skipDefaultCheckout(true) }
    // เพื่อปิดการ checkout อัตโนมัติก่อนเข้า stages (เพราะเรามี checkout scm อยู่แล้ว)
    options { 
        skipDefaultCheckout(true)   // ถ้าเป็น Pipeline from SCM/Multi-branch
    }

    // กำหนด environment variables
    environment {

        // กำหนดค่า Docker Hub credentials ID ที่ตั้งค่าไว้ใน Jenkins
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/express-docker-app"

        // กำหนดค่าสำหรับจำลอง DEV environment บน Local
        DEV_APP_NAME              = "express-app-dev"
        DEV_HOST_PORT             = "3001"

        // กำหนดค่าสำหรับจำลอง PROD environment บน Local
        PROD_APP_NAME             = "express-app-prod"
        PROD_HOST_PORT            = "3000"
    }

    // กำหนด input parameters สำหรับเลือก Action (Build & Deploy หรือ Rollback)
    // และกำหนดค่า ROLLBACK_TAG กับ ROLLBACK_TARGET เมื่อเลือก Rollback
    parameters {
        choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
        string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag ที่ต้องการ (เช่น Git Hash หรือ dev-123)')
        choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือกว่าจะ Rollback ที่ Environment ไหน')
    }

    // กำหนด stages ของ Pipeline
    stages {

        // =================================================================
        // BUILD STAGES: ทำงานเมื่อ ACTION คือ 'Build & Deploy'
        // =================================================================

        // Stage 1: ดึงโค้ดล่าสุดจาก Git
        // ใช้ checkout scm หากใช้ Pipeline from SCM
        // หรือใช้ git url: 'https://github.com/your-username/your-repo.git'
        stage('Checkout') {
            // เงื่อนไข: เมื่อ ACTION คือ 'Build & Deploy' เท่านั้น
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        // Stage 2: ติดตั้ง dependencies และ Run test
        // ใช้ Node.js plugin (ต้องติดตั้ง NodeJS plugin ก่อน) ใน Jenkins หรือ Node.js ใน Docker 
        // ถ้ามี package-lock.json ให้ใช้ npm ci แทน npm install จะเร็วและล็อกเวอร์ชันชัดเจนกว่า
       stage('Install & Test') {
            // เงื่อนไข: เมื่อ ACTION คือ 'Build & Deploy' เท่านั้น
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Running tests inside a consistent Docker environment..."
                 script {
                    docker.image('node:22-alpine').inside {
                        sh '''
                            if [ -f package-lock.json ]; then npm ci; else npm install; fi
                            npm test
                        '''
                    }
                }
            }
        }

        // Stage 3: สร้าง Docker Image
        // ใช้ Docker ที่ติดตั้งบน Jenkins agent (ต้องติดตั้ง Docker plugin ก่อน) ใน Jenkins หรือ Docker ใน Docker
        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag
                    
                    // [ปรับปรุง] ใช้ docker.withRegistry() เพื่อความปลอดภัยและเรียบง่าย
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "--target production .")
                        
                        echo "Pushing images to Docker Hub..."
                        customImage.push()
                        // Push 'latest' tag เฉพาะเมื่อเป็น branch main
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        // =================================================================
        // DEPLOY STAGES: ทำงานเมื่อ ACTION คือ 'Build & Deploy' ตามแต่ละ Branch
        // =================================================================

        // Stage 6: Deploy ไปยังเครื่อง local
        // ดึง image ล่าสุดจาก Docker Hub มาใช้งาน
        // หยุดและลบ container เก่าที่ชื่อ ${APP_NAME} (ถ้ามี)
        // สร้างและรัน container ใหม่จาก image ล่าสุด
        stage('Deploy to DEV (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            } 
            steps {
                script {
                    def deployCmd = """
                            echo "Deploying container ${DEV_APP_NAME} from latest image..."
                            docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker stop ${DEV_APP_NAME} || true
                            docker rm ${DEV_APP_NAME} || true
                            docker run -d --name ${DEV_APP_NAME} -p ${DEV_HOST_PORT}:3000 ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker ps --filter name=${DEV_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                        """
                    sh deployCmd
                }
            }
            // ส่งข้อมูลไปยัง n8n webhook เมื่อ deploy สำเร็จ
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV (Local Docker)', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        // Stage 7: รอการอนุมัติ (Approval) ก่อน Deploy ไปยัง Production
        // เงื่อนไข: เมื่อ ACTION คือ 'Build & Deploy' และ branch คือ 'main'
        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Deploy image tag '${env.IMAGE_TAG}' to PRODUCTION (Local Docker on port ${PROD_HOST_PORT})?"
                }
            }
        }

        // Stage 8: Deploy ไปยังเครื่อง local (Production)
        // ดึง image ล่าสุดจาก Docker Hub มาใช้งาน
        stage('Deploy to PRODUCTION (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            } 
            steps {
                script {
                    def deployCmd = """
                            echo "Deploying container ${PROD_APP_NAME} from latest image..."
                            docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker stop ${PROD_APP_NAME} || true
                            docker rm ${PROD_APP_NAME} || true
                            docker run -d --name ${PROD_APP_NAME} -p ${PROD_HOST_PORT}:3000 ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker ps --filter name=${PROD_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                        """
                    sh deployCmd
                }
            }
            // ส่งข้อมูลไปยัง n8n webhook เมื่อ deploy สำเร็จ
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION (Local Docker)', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        // =================================================================
        // ROLLBACK STAGE: ทำงานเมื่อ ACTION คือ 'Rollback'
        // =================================================================
        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (params.ROLLBACK_TAG.trim().isEmpty()) {
                        error "เมื่อเลือก Rollback กรุณาระบุ 'ROLLBACK_TAG'"
                    }

                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"
                    
                    echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to image: ${imageToDeploy}"
                    
                    def deployCmd = """
                        docker pull ${imageToDeploy}
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                        docker run -d --name ${targetAppName} -p ${targetHostPort}:3000 ${imageToDeploy}
                    """
                    sh(deployCmd)
                }
            }
            post {
                success { 
                    sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
                }
            }
        }
    }

    // กำหนด post actions
    // เช่น การแจ้งเตือนเมื่อ pipeline เสร็จสิ้น
    // สามารถเพิ่มการแจ้งเตือนผ่าน email, Slack, หรืออื่นๆ ได้ตามต้องการ
   post {
        always {
            // ใช้ script block เพื่อให้สามารถใช้เงื่อนไข if ได้
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "Cleaning up Docker images on agent..."
                    // ใช้ try-catch เพื่อให้ pipeline ไม่ล้มเหลวหากลบ image ไม่สำเร็จ
                    try {
                        sh """
                            docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                            docker image rm -f ${DOCKER_REPO}:latest || true
                        """
                    } catch (err) {
                        echo "Could not clean up images, but continuing..."
                    }
                }
            }
        }
        failure {
            // ส่งข้อมูลไปยัง n8n webhook เมื่อ pipeline ล้มเหลว
            sendNotificationToN8n('failed', "Pipeline Failed", 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 5. Push to GitHub

```bash
git add .
git commit -m "Update Docker files"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 6. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 7. แก้ไข app.ts เพื่อทดสอบ

```typescript
.
.
// GET /api/orders
app.get('/api/orders', (_: Request, res: Response) => {
  const orders = [
    { id: 1, userId: 1, productId: 2, quantity: 1 },
    { id: 2, userId: 2, productId: 3, quantity: 2 },
    { id: 3, userId: 1, productId: 1, quantity: 1 },
    { id: 4, userId: 2, productId: 4, quantity: 1 }
  ]
  res.json(orders)
})
.
.
```

#### 8. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "Express-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs (ป้องกันการสร้าง job ซ้ำซ้อนระหว่าง branch กับ PR)
- **Discover pull requests from origin:** The current pull request revision (สร้าง job สำหรับ PR จาก origin)
- **Discover pull requests from forks:** The current pull request revision (สร้าง job สำหรับ PR จาก forks)
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 9. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 10. ทดสอบ API endpoints
```bash
# ทดสอบ DEV environment (port 3001)
curl -X GET http://localhost:3001/api/hello
curl -X GET http://localhost:3001/api/health
curl -X GET http://localhost:3000/api/products
curl -X GET http://localhost:3000/api/orders
```
```bash
# ทดสอบ PROD environment (port 3000)
curl -X GET http://localhost:3000/api/hello
curl -X GET http://localhost:3000/api/health
curl -X GET http://localhost:3000/api/users
curl -X GET http://localhost:3000/api/products
curl -X GET http://localhost:3000/api/orders
```


### Python Flask Jenkins multibranch pipeline

### 🏗️ Project Structure

```
flask-docker-app/
├── 📄 app.py                       # Main Flask application
├── 📄 requirements.txt             # Python dependencies
├── 📁 tests/
│   ├── 📄 __init__.py             # Test package initializer
│   ├── 📄 conftest.py             # Pytest configuration & fixtures
│   └── 📄 test_app.py             # Pytest test suite
├── 📁 __pycache__/                # Python cache files
├── 📁 .pytest_cache/              # Pytest cache
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 main.yml            # GitHub Actions workflow
├── 📄 .dockerignore               # Files to ignore in Docker build
├── 📄 .gitignore                  # Files to ignore in Git
├── 🐳 Dockerfile                   # Docker build configuration
├── 🐳 docker-compose.dev.yml       # Docker Compose for development
├── 🔧 Jenkinsfile                  # Jenkins CI/CD pipeline
├── ⚙️ pytest.ini                  # Pytest configuration
└── 📄 README.md                   # Project documentation
```

#### 1. app.py

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/hello', methods=['GET'])
def hello():
    return jsonify({"message": "Hello from Flask API!"})

@app.route('/api/goodbye', methods=['GET'])
def goodbye():
    return jsonify({"message": "Goodbye from Flask API!"})

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### 2. requirements.txt

```txt
flask==3.0.3
pytest==8.2.2
pytest-flask==1.3.0
```

#### 3. tests/test_app.py

```python
import pytest
from app import app


@pytest.fixture
def client():
    """สร้าง test client สำหรับ Flask app"""
    app.config['TESTING'] = True
    with app.test_client() as client:
        yield client


def test_hello_endpoint(client):
    """ทดสอบ API endpoint /api/hello"""
    response = client.get('/api/hello')
    
    # ตรวจสอบ status code
    assert response.status_code == 200
    
    # ตรวจสอบ response data
    json_data = response.get_json()
    assert json_data is not None
    assert json_data['message'] == 'Hello from Flask API!'


def test_hello_endpoint_method_not_allowed(client):
    """ทดสอบ HTTP method ที่ไม่อนุญาต"""
    response = client.post('/api/hello')
    assert response.status_code == 405  # Method Not Allowed


def test_non_existent_endpoint(client):
    """ทดสอบ endpoint ที่ไม่มีอยู่"""
    response = client.get('/api/notfound')
    assert response.status_code == 404  # Not Found


def test_app_health(client):
    """ทดสอบ health check พื้นฐาน"""
    response = client.get('/api/hello')
    assert response.status_code == 200
    assert response.content_type == 'application/json'
```

#### 4. pytest.ini

```ini
[tool:pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
python_classes = Test*
addopts = -v --tb=short --strict-markers
markers =
    slow: marks tests as slow (deselect with '-m "not slow"')
    integration: marks tests as integration tests
    unit: marks tests as unit tests
```

#### 5. Dockerfile

```dockerfile
# ใช้ Official Python image เป็น base image
FROM python:3.13-slim

# กำหนด Working Directory ภายใน Container
WORKDIR /app

# Copy ไฟล์ requirements.txt เข้าไปก่อน เพื่อใช้ cache layer ของ Docker
COPY requirements.txt .

# ติดตั้ง Dependencies ที่ระบุไว้
RUN pip install --no-cache-dir -r requirements.txt

# Copy โค้ดทั้งหมดในโปรเจกต์เข้าไปใน container
COPY . .

# กำหนด Port ที่ Container จะทำงาน
EXPOSE 5000

# คำสั่งสำหรับรัน Flask Application
CMD ["python", "app.py"]
```

#### 6. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # สร้าง image จาก Dockerfile (โปรเจกต์นี้ไม่มี multi-stage จึงไม่สามารถกำหนด target แบบ builder ได้)
    build:
      context: .
    container_name: flask-app-dev-instance
    ports:
      - "5002:5000" # map port สำหรับ dev (host:container)
    volumes:
      # เชื่อมโค้ดจากเครื่องเราเข้าไปใน container แบบ Real-time เพื่อใช้ auto-reload ของ Flask
      - ./:/app
    environment:
      # เปิดโหมดพัฒนาและเปิด auto-reload
      FLASK_APP: app.py
      FLASK_DEBUG: "1"
      PYTHONDONTWRITEBYTECODE: "1"
    # ติดตั้ง dependencies ทุกครั้งที่รัน (กันเคสแก้ requirements.txt แล้วไม่ rebuild image) และสั่งรัน flask dev server
    command: ["sh", "-c", "pip install --no-cache-dir -r requirements.txt && flask run --host=0.0.0.0 --port=5000"]
```

#### 7. .dockerignore

```dockerignore
# Python cache
__pycache__
*.py[cod]
*$py.class
*.so
.Python

# Virtual environments
venv/
env/
ENV/
.venv

# Testing
.pytest_cache/
.coverage
htmlcov/
*.cover
.hypothesis/

# Environment files
.env
.env.local
.env.*.local

# Git
.git
.gitignore

# Docker
Dockerfile
.dockerignore
docker-compose*.yml

# Documentation
README.md
*.md

# IDE
.vscode
.idea
*.swp
*.swo
.DS_Store

# Logs
*.log
logs/

# Distribution / packaging
dist/
build/
*.egg-info/
```

#### 8. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: ส่ง Notification ไปยัง n8n
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/flask-docker-app"

        // DEV environment
        DEV_APP_NAME              = "flask-app-dev"
        DEV_HOST_PORT             = "5001"

        // PROD environment
        PROD_APP_NAME             = "flask-app-prod"
        PROD_HOST_PORT            = "5000"
    }

    parameters {
        choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
        string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag (เช่น Git Hash หรือ dev-123)')
        choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือก Environment')
    }

    stages {
        // Stage 1: Checkout
        stage('Checkout') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        // Stage 2: Install & Test
        stage('Install & Test') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Running tests inside a consistent Docker environment..."
                script {
                    docker.image('python:3.13-slim').inside {
                        sh '''
                            pip install --no-cache-dir -r requirements.txt
                            pytest -v --tb=short --junitxml=test-results.xml
                        '''
                    }
                }
            }
            post {
                always {
                    junit 'test-results.xml'
                }
            }
        }

        // Stage 3: Build & Push Docker Image
        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag

                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}")

                        echo "Pushing images to Docker Hub..."
                        customImage.push()
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        // Deploy to DEV
        stage('Deploy to DEV (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            }
            steps {
                script {
                    def deployCmd = """
                            echo "Deploying container ${DEV_APP_NAME} from latest image..."
                            docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker stop ${DEV_APP_NAME} || true
                            docker rm ${DEV_APP_NAME} || true
                            docker run -d --name ${DEV_APP_NAME} -p ${DEV_HOST_PORT}:5000 ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker ps --filter name=${DEV_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                        """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV (Local Docker)', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        // Approval for Production
        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Deploy image tag '${env.IMAGE_TAG}' to PRODUCTION (Local Docker on port ${PROD_HOST_PORT})?"
                }
            }
        }

        // Deploy to PROD
        stage('Deploy to PRODUCTION (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                script {
                    def deployCmd = """
                            echo "Deploying container ${PROD_APP_NAME} from latest image..."
                            docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker stop ${PROD_APP_NAME} || true
                            docker rm ${PROD_APP_NAME} || true
                            docker run -d --name ${PROD_APP_NAME} -p ${PROD_HOST_PORT}:5000 ${DOCKER_REPO}:${env.IMAGE_TAG}
                            docker ps --filter name=${PROD_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                        """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION (Local Docker)', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        // Rollback
        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (params.ROLLBACK_TAG.trim().isEmpty()) {
                        error "เมื่อเลือก Rollback กรุณาระบุ 'ROLLBACK_TAG'"
                    }

                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"

                    echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to image: ${imageToDeploy}"

                    def deployCmd = """
                        docker pull ${imageToDeploy}
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                        docker run -d --name ${targetAppName} -p ${targetHostPort}:5000 ${imageToDeploy}
                    """
                    sh(deployCmd)
                }
            }
            post {
                success {
                    script {
                        def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                        def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                        sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "Cleaning up Docker images on agent..."
                    try {
                        sh """
                            docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                            docker image rm -f ${DOCKER_REPO}:latest || true
                        """
                    } catch (err) {
                        echo "Could not clean up images, but continuing..."
                    }
                }
            }
        }
        failure {
            sendNotificationToN8n('failed', "Pipeline Failed", 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 9. Push to GitHub

```bash
git add .
git commit -m "Initial Flask Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 10. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 11. แก้ไข app.py เพื่อทดสอบ

```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/hello', methods=['GET'])
def hello():
    return jsonify({"message": "Hello from Flask API!"})

@app.route('/api/goodbye', methods=['GET'])
def goodbye():
    return jsonify({"message": "Goodbye from Flask API!"})

# เพิ่ม endpoint ใหม่สำหรับทดสอบ
@app.route('/api/users', methods=['GET'])
def users():
    users_list = [
        {"id": 1, "name": "John Doe", "email": "john@example.com"},
        {"id": 2, "name": "Jane Smith", "email": "jane@example.com"},
        {"id": 3, "name": "Bob Johnson", "email": "bob@example.com"}
    ]
    return jsonify(users_list)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

#### 12. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "Flask-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 13. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 14. ทดสอบ API endpoints
```bash
# ทดสอบ DEV environment (port 5001)
curl http://localhost:5001/api/hello
curl http://localhost:5001/api/goodbye
curl http://localhost:5001/api/users

# ทดสอบ PROD environment (port 5000)
curl http://localhost:5000/api/hello
curl http://localhost:5000/api/goodbye
curl http://localhost:5000/api/users
```

### Java Spring Boot Jenkins multibranch pipeline

### 🏗️ Project Structure

```
springboot-docker-app/
├── 📁 src/
│   ├── 📁 main/
│   │   ├── 📁 java/
│   │   │   └── 📁 com/
│   │   │       └── 📁 example/
│   │   │           └── 📁 demo/
│   │   │               ├── 📄 DemoApplication.java      # Main Spring Boot application
│   │   │               ├── 📁 controller/
│   │   │               │   └── 📄 HelloController.java  # REST API Controller
│   │   │               └── 📁 model/
│   │   │                   └── 📄 GreetingRequest.java  # Model class
│   │   └── 📁 resources/
│   │       ├── 📄 application.properties               # Application configuration
│   │       ├── 📁 static/                              # Static resources
│   │       └── 📁 templates/                           # Templates (if using)
│   └── 📁 test/
│       └── 📁 java/
│           └── 📁 com/
│               └── 📁 example/
│                   └── 📁 demo/
│                       ├── 📄 DemoApplicationTests.java                    # Application tests
│                       ├── 📁 controller/
│                       │   └── 📄 HelloControllerTest.java                # Controller unit tests
│                       └── 📁 integration/
│                           └── 📄 HelloControllerIntegrationTest.java     # Integration tests
├── 📁 target/                                          # Maven build output
│   ├── 📄 *.jar                                        # Compiled JAR file
│   ├── 📁 classes/                                     # Compiled classes
│   ├── 📁 surefire-reports/                           # Test reports
│   └── 📁 site/
│       └── 📁 jacoco/                                  # JaCoCo coverage reports
├── 📁 .mvn/
│   └── 📁 wrapper/
│       └── 📄 maven-wrapper.properties                # Maven wrapper configuration
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 main.yml                                # GitHub Actions workflow
├── 📄 .gitignore                                       # Files to ignore in Git
├── 📄 .gitattributes                                   # Git attributes
├── 🐳 Dockerfile                                        # Docker build configuration (Multi-stage)
├── 🐳 docker-compose.dev.yml                            # Docker Compose for development
├── 🔧 Jenkinsfile                                       # Jenkins CI/CD pipeline
├── 📄 pom.xml                                          # Maven project configuration
├── 📄 mvnw                                             # Maven wrapper script (Linux/Mac)
├── 📄 mvnw.cmd                                         # Maven wrapper script (Windows)
├── 📄 HELP.md                                          # Help documentation
└── 📄 README.md                                        # Project documentation
```

#### 1. pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>3.5.6</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.example</groupId>
	<artifactId>demo</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo</name>
	<description>Demo project for Spring Boot</description>
	<properties>
		<java.version>21</java.version>
	</properties>
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
			
			<!-- JaCoCo Plugin สำหรับ Code Coverage -->
			<plugin>
				<groupId>org.jacoco</groupId>
				<artifactId>jacoco-maven-plugin</artifactId>
				<version>0.8.12</version>
				<executions>
					<execution>
						<goals>
							<goal>prepare-agent</goal>
						</goals>
					</execution>
					<execution>
						<id>report</id>
						<phase>test</phase>
						<goals>
							<goal>report</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			
			<!-- Surefire Plugin สำหรับ Unit Tests -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-surefire-plugin</artifactId>
				<version>3.0.0-M9</version>
				<configuration>
					<includes>
						<include>**/*Test.java</include>
						<include>**/*Tests.java</include>
					</includes>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

#### 2. DemoApplication.java

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```

#### 3. HelloController.java

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import java.util.HashMap;
import java.util.Map;

@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public ResponseEntity<Map<String, String>> hello() {
        Map<String, String> response = new HashMap<>();
        response.put("message", "Hello from Spring Boot API!");
        response.put("status", "success");
        return ResponseEntity.ok(response);
    }

    @GetMapping("/hello/{name}")
    public ResponseEntity<Map<String, String>> helloWithName(@PathVariable String name) {
        Map<String, String> response = new HashMap<>();
        response.put("message", "Hello " + name + " from Spring Boot API!");
        response.put("status", "success");
        return ResponseEntity.ok(response);
    }

    @PostMapping("/greet")
    public ResponseEntity<Map<String, String>> greet(@RequestBody Map<String, String> request) {
        String name = request.getOrDefault("name", "World");
        Map<String, String> response = new HashMap<>();
        response.put("greeting", "Greetings " + name + "!");
        response.put("status", "success");
        return ResponseEntity.ok(response);
    }

    @GetMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        Map<String, Object> response = new HashMap<>();
        response.put("status", "UP");
        response.put("timestamp", System.currentTimeMillis());
        response.put("service", "Spring Boot Demo API");
        return ResponseEntity.ok(response);
    }

    @GetMapping("/info")
    public ResponseEntity<Map<String, String>> info() {
        Map<String, String> response = new HashMap<>();
        response.put("app", "Spring Boot Demo Application");
        response.put("version", "1.0.0");
        response.put("description", "A simple Spring Boot application with Docker support.");
        return ResponseEntity.ok(response);
    }
}
```

#### 4. HelloControllerTest.java

```java
package com.example.demo.controller;

import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import java.util.HashMap;
import java.util.Map;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.hamcrest.Matchers.*;

@WebMvcTest(HelloController.class)
class HelloControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    void testHelloEndpoint() throws Exception {
        mockMvc.perform(get("/api/hello"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.message", is("Hello from Spring Boot API!")))
                .andExpect(jsonPath("$.status", is("success")));
    }

    @Test
    void testHelloWithNameEndpoint() throws Exception {
        String name = "John";
        mockMvc.perform(get("/api/hello/{name}", name))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.message", is("Hello " + name + " from Spring Boot API!")))
                .andExpect(jsonPath("$.status", is("success")));
    }

    @Test
    void testGreetEndpoint() throws Exception {
        Map<String, String> request = new HashMap<>();
        request.put("name", "Alice");

        mockMvc.perform(post("/api/greet")
                        .contentType(MediaType.APPLICATION_JSON)
                        .content(objectMapper.writeValueAsString(request)))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.greeting", is("Greetings Alice!")))
                .andExpect(jsonPath("$.status", is("success")));
    }

    @Test
    void testHealthEndpoint() throws Exception {
        mockMvc.perform(get("/api/health"))
                .andExpect(status().isOk())
                .andExpect(content().contentType(MediaType.APPLICATION_JSON))
                .andExpect(jsonPath("$.status", is("UP")))
                .andExpect(jsonPath("$.service", is("Spring Boot Demo API")))
                .andExpect(jsonPath("$.timestamp", notNullValue()));
    }
}
```

#### 5. application.properties

```properties
spring.application.name=demo
```

#### 6. Dockerfile

```dockerfile
# STAGE 1: Build Stage - ใช้ Maven Image เพื่อ Build โปรเจกต์
FROM maven:3.9-eclipse-temurin-21 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# STAGE 2: Run Stage - ใช้ JRE Image ขนาดเล็กเพื่อ Run แอป
FROM eclipse-temurin:21-jre-jammy
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

#### 7. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # ใช้ image จาก Dockerfile แต่ build ถึงแค่ stage 'build' (มี Maven + โค้ด) สำหรับงานพัฒนา
    build:
      context: .
      target: build
    container_name: springboot-app-dev-instance
    ports:
      - "8082:8080" # dev port mapping (host:container)
    volumes:
      # เชื่อมโค้ดแบบ Real-time เพื่อให้แก้แล้ว recompile/restart ได้
      - ./src:/app/src
      - ./pom.xml:/app/pom.xml
      # cache local maven repo ให้ build เร็วขึ้น
      - maven-repo:/root/.m2
    environment:
      SPRING_PROFILES_ACTIVE: dev
      MAVEN_OPTS: -Dmaven.repo.local=/root/.m2/repository
      # ให้ DevTools เฝ้าดูโฟลเดอร์ซอร์สโดยตรง เพื่อ restart เมื่อไฟล์เปลี่ยน
      SPRING_DEVTOOLS_RESTART_ADDITIONAL_PATHS: /app/src/main/java,/app/src/main/resources
    # รัน spring-boot:run พร้อมตัวเฝ้าไฟล์ที่คอมไพล์ใหม่อัตโนมัติเมื่อแก้ไขโค้ด
    command: >-
      sh -c "
      apt-get update && apt-get install -y inotify-tools && \
      ( while inotifywait -r -e modify,create,delete src/main/java src/main/resources; do mvn -q -DskipTests=true compile; done ) & \
      mvn -B -ntp -Dspring-boot.run.addResources -Dspring-boot.run.profiles=dev spring-boot:run
      "

volumes:
  maven-repo:
```

#### 8. .dockerignore

```dockerignore
# Maven build output
target/
!target/*.jar

# Maven wrapper
.mvn/wrapper/maven-wrapper.jar

# IDE files
.idea/
.vscode/
*.iml
*.iws
*.ipr

# Eclipse
.settings/
.classpath
.project

# Git
.git/
.gitignore
.gitattributes

# Docker
Dockerfile
.dockerignore
docker-compose*.yml

# Documentation
README.md
*.md
HELP.md

# Logs
*.log
logs/

# OS
.DS_Store
Thumbs.db

# Test reports
surefire-reports/
jacoco/
```

#### 9. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: ส่ง Notification ไปยัง n8n
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    agent any
    options { skipDefaultCheckout(true) }

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/springboot-docker-app"

        // DEV environment
        DEV_APP_NAME              = "springboot-app-dev"
        DEV_HOST_PORT             = "8081"

        // PROD environment
        PROD_APP_NAME             = "springboot-app-prod"
        PROD_HOST_PORT            = "8080"
    }

    parameters {
        choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
        string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag (เช่น Git Hash หรือ dev-123)')
        choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือก Environment')
    }

    stages {
        
        stage('Checkout & Init') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    checkout scm
                    // ค้นหา Directory ของโปรเจกต์
                    def found = findFiles(glob: '**/pom.xml')
                    if (found.length == 0) {
                        error 'ไม่พบ pom.xml ใน workspace'
                    }
                    // ใช้ Elvis Operator เพื่อจัดการกรณี pom.xml อยู่ที่ root
                    env.PROJECT_DIR = new File(found[0].path).parent ?: '.'
                    echo "PROJECT_DIR set to: '${env.PROJECT_DIR}'"
                }
            }
        }

        stage('Test & Package') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    dir(env.PROJECT_DIR) {
                        echo "Running Maven Test & Package inside Docker..."
                        docker.image('maven:3.9-eclipse-temurin-21').inside {
                            sh 'mvn -B -ntp clean package'
                        }
                    }
                }
            }
            post {
                always {
                    dir(env.PROJECT_DIR) {
                        junit 'target/surefire-reports/*.xml'
                        // JaCoCo Coverage Report
                        publishHTML(target: [
                            allowMissing: true, 
                            reportDir: 'target/site/jacoco', 
                            reportFiles: 'index.html',
                            reportName: 'JaCoCo Coverage', 
                            keepAll: true
                        ])
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag

                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "${env.PROJECT_DIR}")
                        
                        customImage.push()
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy to DEV (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            }
            steps {
                script {
                    sh """
                        echo "Deploying container ${DEV_APP_NAME} from latest image..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${DEV_APP_NAME} || true
                        docker rm ${DEV_APP_NAME} || true
                        docker run -d --name ${DEV_APP_NAME} -p ${DEV_HOST_PORT}:8080 ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${DEV_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                    """
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV (Local Docker)', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Deploy image tag '${env.IMAGE_TAG}' to PRODUCTION (Local Docker on port ${PROD_HOST_PORT})?"
                }
            }
        }

        stage('Deploy to PRODUCTION (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                script {
                    sh """
                        echo "Deploying container ${PROD_APP_NAME} from latest image..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${PROD_APP_NAME} || true
                        docker rm ${PROD_APP_NAME} || true
                        docker run -d --name ${PROD_APP_NAME} -p ${PROD_HOST_PORT}:8080 ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${PROD_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                    """
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION (Local Docker)', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (params.ROLLBACK_TAG.trim().isEmpty()) {
                        error "เมื่อเลือก Rollback กรุณาระบุ 'ROLLBACK_TAG'"
                    }

                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"

                    echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to image: ${imageToDeploy}"

                    sh """
                        docker pull ${imageToDeploy}
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                        docker run -d --name ${targetAppName} -p ${targetHostPort}:8080 ${imageToDeploy}
                    """
                }
            }
            post {
                success {
                    script {
                        def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                        def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                        sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "Cleaning up Docker images on agent..."
                    try {
                        if (env.IMAGE_TAG) {
                            sh """
                                docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                                docker image rm -f ${DOCKER_REPO}:latest || true
                            """
                        } else {
                            echo 'IMAGE_TAG not set, skipping image cleanup.'
                        }
                    } catch (err) {
                        echo "Could not clean up images, but continuing..."
                    }
                }
            }
        }
        failure {
            sendNotificationToN8n('failed', 'Pipeline Failed', 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 10. Push to GitHub

```bash
git add .
git commit -m "Initial Spring Boot Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 11. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 12. แก้ไข HelloController.java เพื่อทดสอบ

```java
package com.example.demo.controller;

import org.springframework.web.bind.annotation.*;
import org.springframework.http.ResponseEntity;
import java.util.HashMap;
import java.util.Map;
import java.util.List;
import java.util.ArrayList;

@RestController
@RequestMapping("/api")
public class HelloController {

    @GetMapping("/hello")
    public ResponseEntity<Map<String, String>> hello() {
        Map<String, String> response = new HashMap<>();
        response.put("message", "Hello from Spring Boot API!");
        response.put("status", "success");
        return ResponseEntity.ok(response);
    }

    @GetMapping("/hello/{name}")
    public ResponseEntity<Map<String, String>> helloWithName(@PathVariable String name) {
        Map<String, String> response = new HashMap<>();
        response.put("message", "Hello " + name + " from Spring Boot API!");
        response.put("status", "success");
        return ResponseEntity.ok(response);
    }

    @PostMapping("/greet")
    public ResponseEntity<Map<String, String>> greet(@RequestBody Map<String, String> request) {
        String name = request.getOrDefault("name", "World");
        Map<String, String> response = new HashMap<>();
        response.put("greeting", "Greetings " + name + "!");
        response.put("status", "success");
        return ResponseEntity.ok(response);
    }

    @GetMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        Map<String, Object> response = new HashMap<>();
        response.put("status", "UP");
        response.put("timestamp", System.currentTimeMillis());
        response.put("service", "Spring Boot Demo API");
        return ResponseEntity.ok(response);
    }

    @GetMapping("/info")
    public ResponseEntity<Map<String, String>> info() {
        Map<String, String> response = new HashMap<>();
        response.put("app", "Spring Boot Demo Application");
        response.put("version", "1.0.0");
        response.put("description", "A simple Spring Boot application with Docker support.");
        return ResponseEntity.ok(response);
    }

    // เพิ่ม endpoint ใหม่สำหรับทดสอบ
    @GetMapping("/products")
    public ResponseEntity<List<Map<String, Object>>> products() {
        List<Map<String, Object>> products = new ArrayList<>();
        
        Map<String, Object> product1 = new HashMap<>();
        product1.put("id", 1);
        product1.put("name", "Laptop");
        product1.put("price", 25000.00);
        products.add(product1);
        
        Map<String, Object> product2 = new HashMap<>();
        product2.put("id", 2);
        product2.put("name", "Mouse");
        product2.put("price", 500.00);
        products.add(product2);
        
        Map<String, Object> product3 = new HashMap<>();
        product3.put("id", 3);
        product3.put("name", "Keyboard");
        product3.put("price", 1200.00);
        products.add(product3);
        
        return ResponseEntity.ok(products);
    }
}
```

#### 13. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "SpringBoot-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 14. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 15. ทดสอบ API endpoints
```bash
# ทดสอบ DEV environment (port 8081)
curl http://localhost:8081/api/hello
curl http://localhost:8081/api/hello/John
curl http://localhost:8081/api/health
curl http://localhost:8081/api/info
curl http://localhost:8081/api/products

# ส่ง POST request
curl -X POST http://localhost:8081/api/greet \
  -H "Content-Type: application/json" \
  -d '{"name":"Alice"}'

# ทดสอบ PROD environment (port 8080)
curl http://localhost:8080/api/hello
curl http://localhost:8080/api/hello/Jane
curl http://localhost:8080/api/health
curl http://localhost:8080/api/info
curl http://localhost:8080/api/products

# ส่ง POST request
curl -X POST http://localhost:8080/api/greet \
  -H "Content-Type: application/json" \
  -d '{"name":"Bob"}'
```

#### 16. ดู JaCoCo Code Coverage Report
- เข้าไปที่ Jenkins Job ที่รันเสร็จแล้ว
- คลิกที่ "JaCoCo Coverage" ใน sidebar
- ดูรายงาน Code Coverage ที่แสดงเปอร์เซ็นต์ของโค้ดที่ถูก test ครอบคลุม


### NextJS Jenkins multibranch pipeline

### 🏗️ Project Structure

```
nextjs-docker-app/
├── 📁 src/
│   └── 📁 app/
│       ├── 📄 page.tsx                     # Home page component
│       ├── 📄 layout.tsx                   # Root layout component
│       ├── 📄 globals.css                  # Global styles
│       └── 📄 favicon.ico                  # Favicon
├── 📁 public/                              # Static assets
│   ├── 📄 file.svg
│   ├── 📄 globe.svg
│   ├── 📄 next.svg
│   ├── 📄 vercel.svg
│   └── 📄 window.svg
├── 📁 components/                          # Reusable components
├── 📁 .next/                               # Next.js build output
│   ├── 📄 build-manifest.json
│   ├── 📄 routes-manifest.json
│   ├── 📁 cache/                           # Build cache
│   ├── 📁 server/                          # Server components
│   └── 📁 static/                          # Static assets
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 main.yml                    # GitHub Actions workflow
├── 📄 .dockerignore                        # Files to ignore in Docker build
├── 📄 .gitignore                           # Files to ignore in Git
├── 🐳 Dockerfile                            # Docker build configuration (Multi-stage)
├── 🐳 docker-compose.dev.yml                # Docker Compose for development
├── 🔧 Jenkinsfile                           # Jenkins CI/CD pipeline
├── 📄 package.json                         # Node.js project configuration
├── 📄 next.config.ts                       # Next.js configuration
├── 📄 next-env.d.ts                        # Next.js TypeScript declarations
├── 📄 tsconfig.json                        # TypeScript configuration
├── 📄 postcss.config.mjs                   # PostCSS configuration
├── 📄 eslint.config.mjs                    # ESLint configuration
└── 📄 README.md                            # Project documentation
```

#### 1. package.json

```json
{
  "name": "nextjs-docker-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --port 4002",
    "build": "next build",
    "start": "next start",
    "lint": "eslint"
  },
  "dependencies": {
    "react": "19.1.0",
    "react-dom": "19.1.0",
    "next": "15.5.5"
  },
  "devDependencies": {
    "typescript": "^5",
    "@types/node": "^20",
    "@types/react": "^19",
    "@types/react-dom": "^19",
    "@tailwindcss/postcss": "^4",
    "tailwindcss": "^4",
    "eslint": "^9",
    "eslint-config-next": "15.5.5",
    "@eslint/eslintrc": "^3"
  }
}
```

#### 2. src/app/page.tsx

```tsx
export default function Home() {
  return (
    <div className="font-sans grid grid-rows-[20px_1fr_20px] items-center justify-items-center min-h-screen p-8 pb-20 gap-16 sm:p-20">
      <h1>Hello Next.js!</h1>
      
      <p className="text-center max-w-lg">
        This is a minimal Next.js app deployed with Docker. You can start editing the code in <code>src/app/page.tsx</code>.
      </p>

      <p className="text-center max-w-lg">
        This app is a great starting point for building modern web applications with Next.js and Docker.
      </p>

      <footer className="text-xs text-center text-gray-500">
        Deployed with ❤️ using Docker
      </footer>
    </div>
  )
}
```

#### 3. src/app/layout.tsx

```tsx
import type { Metadata } from "next";
import { Geist, Geist_Mono } from "next/font/google";
import "./globals.css";

const geistSans = Geist({
  variable: "--font-geist-sans",
  subsets: ["latin"],
});

const geistMono = Geist_Mono({
  variable: "--font-geist-mono",
  subsets: ["latin"],
});

export const metadata: Metadata = {
  title: "Create Next App",
  description: "Generated by create next app",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body
        className={`${geistSans.variable} ${geistMono.variable} antialiased`}
      >
        {children}
      </body>
    </html>
  );
}
```

#### 4. next.config.ts

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;
```

#### 5. Dockerfile

```dockerfile
# ==============================
# Next.js Multi-stage Dockerfile
# ตามแนวทางจาก express-docker-app
# ==============================

# Build stage - ใช้สำหรับ development/testing และ build โปรเจกต์
FROM node:22-alpine AS builder

WORKDIR /app

# ใช้ cache layer: ติดตั้ง deps ก่อน
COPY package*.json ./
RUN if [ -f package-lock.json ]; then npm ci; else npm install; fi

# คัดลอกซอร์สทั้งหมด
COPY . .

# Build โปรเจกต์ (สร้าง .next)
RUN npm run build


# Production stage - สำหรับ production deployment
FROM node:22-alpine AS production

ENV NODE_ENV=production \
	NEXT_TELEMETRY_DISABLED=1

WORKDIR /app

# คัดลอก package files และติดตั้งเฉพาะ production deps
COPY package*.json ./
RUN if [ -f package-lock.json ]; then npm ci --only=production; else npm install --omit=dev; fi \
	&& npm cache clean --force

# คัดลอกไฟล์ build และ assets ที่จำเป็นจาก builder stage
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/next.config.* ./  
COPY --from=builder /app/tsconfig*.json ./  

# เปิดพอร์ตแอป
EXPOSE 3000

# ใช้สคริปต์ start ของโปรเจกต์ (ปกติคือ `next start -p 3000`)
CMD ["npm", "start"]
```

#### 6. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # ใช้แค่ stage 'builder' เพื่อให้มี devDependencies ครบใน dev mode
    build:
      context: .
      target: builder
    container_name: next-app-dev-instance
    ports:
      - "4002:4002"
    volumes:
      # เชื่อมโค้ดแบบ Real-time (App Router ใช้โฟลเดอร์ ./src)
      - ./src:/app/src
      - ./public:/app/public
      - ./package.json:/app/package.json
      - ./package-lock.json:/app/package-lock.json
      - ./tsconfig.json:/app/tsconfig.json
      # ไม่ map node_modules เพื่อใช้ของใน container
      - /app/node_modules
    environment:
      NEXT_TELEMETRY_DISABLED: "1"
      WATCHPACK_POLLING: "true"
      WATCHPACK_POLLING_INTERVAL: "1000"
      CHOKIDAR_USEPOLLING: "true"
      CHOKIDAR_INTERVAL: "1000"
    command: sh -c "npm run dev -- --hostname 0.0.0.0"
```

#### 7. .dockerignore

```dockerignore
# Dependencies
node_modules
npm-debug.log*
pnpm-lock.yaml
yarn.lock

# Build outputs
.next
out
build
dist

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Testing
coverage
*.lcov

# Git
.git
.gitignore

# Docker
Dockerfile
.dockerignore

# Documentation
README.md
*.md

# IDE
.vscode
.idea
*.swp
*.swo

# OS
.DS_Store
Thumbs.db

# Logs
logs
*.log

# Temporary files
.tmp
.temp
```

#### 8. Jenkinsfile

```groovy
// ================================================================
// Jenkins Pipeline for Next.js (App Router) — follows Express pattern
// - Checkout
// - Install & (conditionally) Test in Node Docker
// - Build & Push Docker image (production target)
// - Deploy to DEV on develop branch
// - Approval + Deploy to PROD on main branch
// - Rollback support
// - n8n webhook notifications
// ================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
	script {
		withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
			def payload = [
				project  : env.JOB_NAME,
				stage    : stageName,
				status   : status,
				build    : env.BUILD_NUMBER,
				image    : "${env.DOCKER_REPO}:${imageTag}",
				container: containerName,
				url      : "http://localhost:${hostPort}/",
				timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
			]
			def body = groovy.json.JsonOutput.toJson(payload)
			try {
				httpRequest acceptType: 'APPLICATION_JSON',
							contentType: 'APPLICATION_JSON',
							httpMode: 'POST',
							requestBody: body,
							url: N8N_WEBHOOK_URL,
							validResponseCodes: '200:299'
				echo "n8n webhook (${status}) sent successfully."
			} catch (err) {
				echo "Failed to send n8n webhook (${status}): ${err}"
			}
		}
	}
}

pipeline {
	agent any

	options {
		skipDefaultCheckout(true)
	}

	environment {
		DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
		DOCKER_REPO               = 'iamsamitdev/nextjs-docker-app'
		DEV_APP_NAME              = 'nextjs-app-dev'
		DEV_HOST_PORT             = '4001'
		PROD_APP_NAME             = 'nextjs-app-prod'
		PROD_HOST_PORT            = '4000'
	}

	parameters {
		choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
		string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag ที่ต้องการ (เช่น Git Hash หรือ dev-123)')
		choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือกว่าจะ Rollback ที่ Environment ไหน')
	}

	stages {
		stage('Checkout') {
			when { expression { params.ACTION == 'Build & Deploy' } }
			steps {
				echo 'Checking out code...'
				checkout scm
			}
		}

		stage('Install & Test') {
			when { expression { params.ACTION == 'Build & Deploy' } }
			steps {
				echo 'Installing dependencies and running tests (if present) inside Node Docker...'
				script {
					docker.image('node:22-alpine').inside {
						sh '''
							set -eux
							if [ -f package-lock.json ]; then npm ci; else npm install; fi
							if node -e "const fs=require('fs'); try { const p=JSON.parse(fs.readFileSync('package.json','utf8')); process.exit(p.scripts && p.scripts.test ? 0 : 1); } catch (err) { process.exit(1); }"; then
								npm test
							else
								echo "No test script found. Running lint (non-blocking)."
								npm run -s lint || true
							fi
						'''
					}
				}
			}
		}

		stage('Build & Push Docker Image') {
			when { expression { params.ACTION == 'Build & Deploy' } }
			steps {
				script {
					def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
					env.IMAGE_TAG = imageTag

					docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
						echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
						
						def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "--target production .")
						
						echo 'Pushing images to Docker Hub...'
						customImage.push()
						if (env.BRANCH_NAME == 'main') {
							customImage.push('latest')
						}
					}
				}
			}
		}

		stage('Deploy to DEV (Local Docker)') {
			when {
				expression { params.ACTION == 'Build & Deploy' }
				branch 'develop'
			}
			steps {
				script {
					def deployCmd = """
						echo "Deploying container ${DEV_APP_NAME}..."
						docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
						docker stop ${DEV_APP_NAME} || true
						docker rm ${DEV_APP_NAME} || true
						docker run -d --name ${DEV_APP_NAME} -p ${DEV_HOST_PORT}:3000 ${DOCKER_REPO}:${env.IMAGE_TAG}
						docker ps --filter name=${DEV_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
					"""
					sh deployCmd
				}
			}
			post {
				success {
					sendNotificationToN8n('success', 'Deploy to DEV (Local Docker)', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
				}
			}
		}

		stage('Approval for Production') {
			when {
				expression { params.ACTION == 'Build & Deploy' }
				branch 'main'
			}
			steps {
				timeout(time: 1, unit: 'HOURS') {
					input message: "Deploy image tag '${env.IMAGE_TAG}' to PRODUCTION (Local Docker on port ${PROD_HOST_PORT})?"
				}
			}
		}

		stage('Deploy to PRODUCTION (Local Docker)') {
			when {
				expression { params.ACTION == 'Build & Deploy' }
				branch 'main'
			}
			steps {
				script {
					def deployCmd = """
						echo "Deploying container ${PROD_APP_NAME}..."
						docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
						docker stop ${PROD_APP_NAME} || true
						docker rm ${PROD_APP_NAME} || true
						docker run -d --name ${PROD_APP_NAME} -p ${PROD_HOST_PORT}:3000 ${DOCKER_REPO}:${env.IMAGE_TAG}
						docker ps --filter name=${PROD_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
					"""
					sh deployCmd
				}
			}
			post {
				success {
					sendNotificationToN8n('success', 'Deploy to PRODUCTION (Local Docker)', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
				}
			}
		}

		stage('Execute Rollback') {
			when { expression { params.ACTION == 'Rollback' } }
			steps {
				script {
					if (params.ROLLBACK_TAG.trim().isEmpty()) {
						error "เมื่อเลือก Rollback กรุณาระบุ 'ROLLBACK_TAG'"
					}

					def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
					def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
					def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"

					echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to image: ${imageToDeploy}"
					
					def deployCmd = """
						docker pull ${imageToDeploy}
						docker stop ${targetAppName} || true
						docker rm ${targetAppName} || true
						docker run -d --name ${targetAppName} -p ${targetHostPort}:3000 ${imageToDeploy}
					"""
					sh(deployCmd)
				}
			}
			post {
				success {
					script {
						def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
						def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
						sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
					}
				}
			}
		}
	}

	post {
		always {
			script {
				if (params.ACTION == 'Build & Deploy' && env.IMAGE_TAG) {
					echo 'Cleaning up Docker images on agent...'
					try {
						sh """
							docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
							docker image rm -f ${DOCKER_REPO}:latest || true
						"""
					} catch (err) {
						echo 'Could not clean up images, but continuing...'
					}
				}
			}
		}
		failure {
			sendNotificationToN8n('failed', 'Pipeline Failed', 'N/A', 'N/A', 'N/A')
		}
	}
}
```

#### 9. Push to GitHub

```bash
git add .
git commit -m "Initial Next.js Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 10. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 11. แก้ไข src/app/page.tsx เพื่อทดสอบ

```tsx
export default function Home() {
  return (
    <div className="font-sans grid grid-rows-[20px_1fr_20px] items-center justify-items-center min-h-screen p-8 pb-20 gap-16 sm:p-20">
      <h1 className="text-4xl font-bold">Hello Next.js with Docker!</h1>
      
      <main className="flex flex-col gap-8 items-center">
        <p className="text-center max-w-lg text-lg">
          This is a Next.js app deployed with Docker and Jenkins CI/CD pipeline.
        </p>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-4 max-w-2xl">
          <div className="p-6 border rounded-lg">
            <h2 className="text-xl font-semibold mb-2">⚡ Fast Refresh</h2>
            <p className="text-sm text-gray-600">
              Edit <code className="bg-gray-100 px-2 py-1 rounded">src/app/page.tsx</code> and see changes instantly.
            </p>
          </div>

          <div className="p-6 border rounded-lg">
            <h2 className="text-xl font-semibold mb-2">🐳 Docker Ready</h2>
            <p className="text-sm text-gray-600">
              Optimized multi-stage Dockerfile for development and production.
            </p>
          </div>

          <div className="p-6 border rounded-lg">
            <h2 className="text-xl font-semibold mb-2">🚀 CI/CD Pipeline</h2>
            <p className="text-sm text-gray-600">
              Automated deployment with Jenkins multibranch pipeline.
            </p>
          </div>

          <div className="p-6 border rounded-lg">
            <h2 className="text-xl font-semibold mb-2">📦 Production Ready</h2>
            <p className="text-sm text-gray-600">
              Separate DEV and PROD environments with rollback support.
            </p>
          </div>
        </div>
      </main>

      <footer className="text-xs text-center text-gray-500">
        Deployed with ❤️ using Docker & Jenkins
      </footer>
    </div>
  )
}
```

#### 12. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "NextJS-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 13. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 14. ทดสอบ Application
```bash
# ทดสอบ DEV environment (port 4001)
# เปิดเบราว์เซอร์
http://localhost:4001

# ทดสอบ PROD environment (port 4000)
# เปิดเบราว์เซอร์
http://localhost:4000
```

#### 15. ทดสอบการทำงานของ Hot Reload (Development Mode)
```bash
# รัน docker-compose สำหรับ development
cd nextjs-docker-app
docker-compose -f docker-compose.dev.yml up

# แก้ไขไฟล์ src/app/page.tsx
# บันทึกไฟล์ แล้วดูการเปลี่ยนแปลงใน browser โดยอัตโนมัติ

# เปิดเบราว์เซอร์ไปที่
http://localhost:4002
```

#### 16. คำสั่งที่เป็นประโยชน์

```bash
# ดู logs ของ container
docker logs nextjs-app-dev
docker logs nextjs-app-prod

# เข้าไปใน container
docker exec -it nextjs-app-dev sh
docker exec -it nextjs-app-prod sh

# ตรวจสอบ container ที่กำลังรัน
docker ps

# หยุด container
docker stop nextjs-app-dev nextjs-app-prod

# ลบ container
docker rm nextjs-app-dev nextjs-app-prod

# ดู Docker images
docker images | grep nextjs-docker-app

# ลบ Docker image
docker rmi iamsamitdev/nextjs-docker-app:latest
```

### .NET Core Jenkins multibranch pipeline

### 🏗️ Project Structure

```
dotnet-docker-app/
├── 📄 Program.cs                           # Main application entry point
├── 📄 dotnet-docker-app.csproj            # .NET project file
├── 📄 appsettings.json                    # Application settings
├── 📄 appsettings.Development.json        # Development settings
├── 📄 dotnet-docker-app.http              # HTTP requests for testing
├── 📁 Properties/
│   └── 📄 launchSettings.json             # Launch settings
├── 📁 bin/                                 # Binary output (compiled)
│   └── 📁 Debug/
│       └── 📁 net9.0/
│           ├── 📄 dotnet-docker-app.dll
│           ├── 📄 dotnet-docker-app.exe
│           ├── 📄 dotnet-docker-app.pdb
│           └── 📄 *.json
├── 📁 obj/                                 # Object files (intermediate)
│   ├── 📄 project.assets.json
│   └── 📁 Debug/
├── 📄 .dockerignore                        # Files to ignore in Docker build
├── 📄 .gitignore                           # Files to ignore in Git
├── 🐳 Dockerfile                            # Docker build configuration (Multi-stage)
├── 🐳 docker-compose.dev.yml                # Docker Compose for development
├── 🔧 Jenkinsfile                           # Jenkins CI/CD pipeline
└── 📄 README.md                            # Project documentation
```

#### 1. dotnet-docker-app.csproj

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net9.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <RootNamespace>dotnet_docker_app</RootNamespace>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="9.0.9" />
  </ItemGroup>

</Project>
```

#### 2. Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring OpenAPI at https://aka.ms/aspnet/openapi
builder.Services.AddOpenApi();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection();

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/", () => "Hello World! Welcome to .NET on Docker!");

app.MapGet("/ping", () => "pong");

app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast");

app.Run();

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```

#### 3. appsettings.json

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

#### 4. Dockerfile

```dockerfile
# Stage 1: Base SDK (for development)
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS base
WORKDIR /src

# Install dotnet tools for hot reload
RUN dotnet tool install --global dotnet-ef
ENV PATH="${PATH}:/root/.dotnet/tools"

# Copy csproj and restore dependencies
COPY ["dotnet-docker-app.csproj", "./"]
RUN dotnet restore "dotnet-docker-app.csproj"

# Stage 2: Build
FROM base AS build
WORKDIR /src

# Copy everything else and build
COPY . .
RUN dotnet build "dotnet-docker-app.csproj" -c Release -o /app/build

# Stage 3: Publish
FROM build AS publish
RUN dotnet publish "dotnet-docker-app.csproj" -c Release -o /app/publish /p:UseAppHost=false

# Stage 4: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS final
WORKDIR /app

# Expose port
EXPOSE 8080
EXPOSE 8081

# Copy published files from publish stage
COPY --from=publish /app/publish .

# Set environment variables
ENV ASPNETCORE_URLS=http://+:8080
ENV ASPNETCORE_ENVIRONMENT=Production

# Run the application
ENTRYPOINT ["dotnet", "dotnet-docker-app.dll"]
```

#### 5. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # สร้าง image จาก Dockerfile แต่จะใช้แค่ 'base' stage เป็นฐาน
    # เพื่อให้มี SDK ครบสำหรับ development และ hot reload
    build:
      context: .
      target: base # <-- บอกให้ build ถึงแค่ stage 'base' ที่มี SDK
    container_name: dotnet-app-dev-instance
    ports:
      - "6002:5262" # HTTP port
      - "7100:7100"  # HTTPS port (optional)
    volumes:
      # เชื่อมโค้ดจากเครื่องเราเข้าไปใน container แบบ Real-time
      - ./Program.cs:/src/Program.cs
      - ./appsettings.json:/src/appsettings.json
      - ./appsettings.Development.json:/src/appsettings.Development.json
      - ./dotnet-docker-app.csproj:/src/dotnet-docker-app.csproj
      - ./Properties:/src/Properties
      # ไม่ต้อง map obj และ bin เพื่อให้ใช้ของใน container
      - /src/obj
      - /src/bin
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=http://+:5262
      - DOTNET_USE_POLLING_FILE_WATCHER=true # เปิดใช้ file watcher สำหรับ hot reload
      - DOTNET_WATCH_RESTART_ON_RUDE_EDIT=true
    # สั่งให้รันด้วย dotnet watch เพื่อ hot reload
    command: dotnet watch run --no-restore --urls "http://+:5262"
    stdin_open: true
    tty: true
```

#### 6. .dockerignore

```dockerignore
# .NET Build artifacts
bin/
obj/
out/

# Visual Studio / VSCode
.vs/
.vscode/
*.user
*.suo
*.userosscache
*.sln.docstates

# Build results
[Dd]ebug/
[Rr]elease/
x64/
x86/
[Aa][Rr][Mm]/
[Aa][Rr][Mm]64/
bld/
[Bb]in/
[Oo]bj/
[Ll]og/
[Ll]ogs/

# Test Results
[Tt]est[Rr]esult*/
[Bb]uild[Ll]og.*
*.trx
*.coverage
*.coveragexml

# NuGet Packages
*.nupkg
*.snupkg
packages/
.nuget/

# Node modules (if any)
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Docker
.dockerignore
Dockerfile
docker-compose*.yml
*.dockerfile

# Git
.git
.gitignore
.gitattributes

# Documentation
README.md
*.md

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Environment files
.env
.env.local
.env.development
.env.production

# IDE
*.swp
*.swo
.idea/
```

#### 7. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: สร้างฟังก์ชันสำหรับส่ง Notification ไปยัง n8n
// การสร้างฟังก์ชันช่วยลดการเขียนโค้ดซ้ำซ้อน (DRY Principle)
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/weatherforecast",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    agent any

    options { 
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/dotnet-docker-app"

        // DEV environment
        DEV_APP_NAME              = "dotnet-app-dev"
        DEV_HOST_PORT             = "6001"

        // PROD environment
        PROD_APP_NAME             = "dotnet-app-prod"
        PROD_HOST_PORT            = "6000"
    }

    parameters {
        choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
        string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag ที่ต้องการ (เช่น Git Hash หรือ dev-123)')
        choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือกว่าจะ Rollback ที่ Environment ไหน')
    }

    stages {

        // Stage 1: Checkout
        stage('Checkout') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        // Stage 2: Restore & Test
        stage('Restore & Test') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Running tests inside a consistent Docker environment..."
                script {
                    docker.image('mcr.microsoft.com/dotnet/sdk:9.0').inside {
                        sh '''
                            dotnet restore
                            dotnet build --no-restore -c Release
                            dotnet test --no-build --verbosity normal -c Release
                        '''
                    }
                }
            }
        }

        // Stage 3: Build & Push Docker Image
        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag
                    
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "--target final .")
                        
                        echo "Pushing images to Docker Hub..."
                        customImage.push()
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        // Deploy to DEV
        stage('Deploy to DEV (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            } 
            steps {
                script {
                    def deployCmd = """
                        echo "Deploying container ${DEV_APP_NAME} from latest image..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${DEV_APP_NAME} || true
                        docker rm ${DEV_APP_NAME} || true
                        docker run -d --name ${DEV_APP_NAME} \
                            -p ${DEV_HOST_PORT}:8080 \
                            -e ASPNETCORE_ENVIRONMENT=Development \
                            -e ASPNETCORE_URLS=http://+:8080 \
                            ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${DEV_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                    """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV (Local Docker)', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        // Approval for Production
        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Deploy image tag '${env.IMAGE_TAG}' to PRODUCTION (Local Docker on port ${PROD_HOST_PORT})?"
                }
            }
        }

        // Deploy to PROD
        stage('Deploy to PRODUCTION (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            } 
            steps {
                script {
                    def deployCmd = """
                        echo "Deploying container ${PROD_APP_NAME} from latest image..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${PROD_APP_NAME} || true
                        docker rm ${PROD_APP_NAME} || true
                        docker run -d --name ${PROD_APP_NAME} \
                            -p ${PROD_HOST_PORT}:8080 \
                            -e ASPNETCORE_ENVIRONMENT=Production \
                            -e ASPNETCORE_URLS=http://+:8080 \
                            ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${PROD_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                    """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION (Local Docker)', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        // Rollback
        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (params.ROLLBACK_TAG.trim().isEmpty()) {
                        error "เมื่อเลือก Rollback กรุณาระบุ 'ROLLBACK_TAG'"
                    }

                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def targetEnv = (params.ROLLBACK_TARGET == 'dev') ? 'Development' : 'Production'
                    def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"
                    
                    echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to image: ${imageToDeploy}"
                    
                    def deployCmd = """
                        docker pull ${imageToDeploy}
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                        docker run -d --name ${targetAppName} \
                            -p ${targetHostPort}:8080 \
                            -e ASPNETCORE_ENVIRONMENT=${targetEnv} \
                            -e ASPNETCORE_URLS=http://+:8080 \
                            ${imageToDeploy}
                    """
                    sh(deployCmd)
                }
            }
            post {
                success {
                    script {
                        def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                        def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                        sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "Cleaning up Docker images on agent..."
                    try {
                        sh """
                            docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                            docker image rm -f ${DOCKER_REPO}:latest || true
                        """
                    } catch (err) {
                        echo "Could not clean up images, but continuing..."
                    }
                }
            }
        }
        failure {
            sendNotificationToN8n('failed', "Pipeline Failed", 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 8. Push to GitHub

```bash
git add .
git commit -m "Initial .NET Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 9. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 10. แก้ไข Program.cs เพื่อทดสอบ

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddOpenApi();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
}

app.UseHttpsRedirection();

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/", () => "Hello World! Welcome to .NET 9 on Docker!");

app.MapGet("/ping", () => "pong");

app.MapGet("/api/health", () => new
{
    status = "Healthy",
    timestamp = DateTime.UtcNow,
    service = ".NET Docker App",
    version = "1.0.0"
});

app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast");

// เพิ่ม endpoint ใหม่สำหรับทดสอบ
app.MapGet("/api/products", () =>
{
    var products = new[]
    {
        new { Id = 1, Name = "Laptop", Price = 25000.00M, InStock = true },
        new { Id = 2, Name = "Mouse", Price = 500.00M, InStock = true },
        new { Id = 3, Name = "Keyboard", Price = 1200.00M, InStock = false },
        new { Id = 4, Name = "Monitor", Price = 8000.00M, InStock = true }
    };
    return Results.Ok(products);
})
.WithName("GetProducts");

app.Run();

record WeatherForecast(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}
```

#### 11. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "DotNet-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 12. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 13. ทดสอบ API endpoints
```bash
# ทดสอบ DEV environment (port 6001)
curl http://localhost:6001/
curl http://localhost:6001/ping
curl http://localhost:6001/api/health
curl http://localhost:6001/weatherforecast
curl http://localhost:6001/api/products

# ทดสอบ PROD environment (port 6000)
curl http://localhost:6000/
curl http://localhost:6000/ping
curl http://localhost:6000/api/health
curl http://localhost:6000/weatherforecast
curl http://localhost:6000/api/products
```

#### 14. ทดสอบการทำงานของ Hot Reload (Development Mode)
```bash
# รัน docker-compose สำหรับ development
cd dotnet-docker-app
docker-compose -f docker-compose.dev.yml up

# แก้ไขไฟล์ Program.cs
# บันทึกไฟล์ แล้ว dotnet watch จะ restart application อัตโนมัติ

# เปิดเบราว์เซอร์ไปที่
http://localhost:6002
```

#### 15. คำสั่งที่เป็นประโยชน์

```bash
# ดู logs ของ container
docker logs dotnet-app-dev
docker logs dotnet-app-prod

# เข้าไปใน container
docker exec -it dotnet-app-dev sh
docker exec -it dotnet-app-prod sh

# ตรวจสอบ container ที่กำลังรัน
docker ps

# หยุด container
docker stop dotnet-app-dev dotnet-app-prod

# ลบ container
docker rm dotnet-app-dev dotnet-app-prod

# ดู Docker images
docker images | grep dotnet-docker-app

# ลบ Docker image
docker rmi iamsamitdev/dotnet-docker-app:latest

# รัน dotnet commands ใน container
docker exec -it dotnet-app-dev dotnet --version
docker exec -it dotnet-app-dev dotnet --info
```

#### 16. ข้อมูลเพิ่มเติมเกี่ยวกับ .NET 9

- **.NET 9** เป็นเวอร์ชันล่าสุดของ .NET (ณ เดือนตุลาคม 2025)
- รองรับ **C# 13** และ **ASP.NET Core 9**
- มี **Performance improvements** และ **New features** มากมาย
- ใช้ **Minimal APIs** สำหรับสร้าง REST APIs อย่างรวดเร็ว
- รองรับ **OpenAPI/Swagger** โดยตรง
- ใช้ **Hot Reload** ในโหมด Development
- Docker images ขนาดเล็กกว่าเดิมด้วย **Alpine-based images**

### Go Fiber Jenkins multibranch pipeline

### 🏗️ Project Structure

```
fiber-docker-app/
├── 📄 main.go                          # Main Fiber application
├── 📄 go.mod                           # Go module file
├── 📄 go.sum                           # Go dependencies checksums
├── 📄 .air.toml                        # Air hot reload configuration
├── 📄 .dockerignore                    # Files to ignore in Docker build
├── 📄 .gitignore                       # Files to ignore in Git
├── 🐳 Dockerfile                        # Docker build configuration (Multi-stage)
├── 🐳 docker-compose.dev.yml            # Docker Compose for development
├── 🔧 Jenkinsfile                       # Jenkins CI/CD pipeline
├── 📁 tmp/                             # Air temporary files
│   └── 📄 main                         # Compiled binary for development
└── 📄 README.md                        # Project documentation
```

#### 1. main.go

```go
package main

import (
	"log"
	"os"
	"os/signal"
	"syscall"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/logger"
	"github.com/gofiber/fiber/v2/middleware/recover"
)

func main() {
	// Create Fiber instance with custom config
	app := fiber.New(fiber.Config{
		AppName:          "Fiber Docker App v1.0.0",
		ServerHeader:     "Fiber",
		ReadBufferSize:   16384,           // 16KB - เพิ่มขนาด buffer สำหรับ request headers
		WriteBufferSize:  16384,           // 16KB - เพิ่มขนาด buffer สำหรับ response
		BodyLimit:        4 * 1024 * 1024, // 4MB - จำกัดขนาด request body
		DisableKeepalive: false,           // เปิดใช้ keep-alive
	})

	// Middleware
	app.Use(logger.New())  // Logging middleware
	app.Use(recover.New()) // Recover from panics

	// Routes
	app.Get("/", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"message": "Hello World 🌈",
			"status":  "success",
			"app":     "Fiber Docker App",
		})
	})

	app.Get("/ping", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"message": "Pong 🏓",
			"status":  "success",
		})
	})

	app.Get("/health", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"status": "ok",
			"uptime": "running",
		})
	})

	// Get port from environment or use default
	port := os.Getenv("PORT")
	if port == "" {
		port = "9000"
	}

	// Channel to listen for interrupt signals for graceful shutdown
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, os.Interrupt, syscall.SIGTERM)

	// Start server in a goroutine so we can listen for shutdown signals
	go func() {
		log.Printf("🚀 Server is starting on port %s...", port)
		log.Printf("🌐 Visit: http://localhost:%s", port)
		if err := app.Listen(":" + port); err != nil {
			log.Printf("❌ Error starting server: %v", err)
		}
	}()

	// Wait for interrupt signal to gracefully shutdown the server
	<-quit
	log.Println("\n🛑 Shutting down server...")

	// Graceful shutdown
	if err := app.Shutdown(); err != nil {
		log.Printf("❌ Error during shutdown: %v", err)
	}

	log.Println("✅ Server stopped gracefully")
}
```

#### 2. go.mod

```go
module fiber-docker-app

go 1.24.2

require github.com/gofiber/fiber/v2 v2.52.9

require (
	github.com/andybalholm/brotli v1.1.0 // indirect
	github.com/google/uuid v1.6.0 // indirect
	github.com/klauspost/compress v1.17.9 // indirect
	github.com/mattn/go-colorable v0.1.13 // indirect
	github.com/mattn/go-isatty v0.0.20 // indirect
	github.com/mattn/go-runewidth v0.0.16 // indirect
	github.com/rivo/uniseg v0.2.0 // indirect
	github.com/valyala/bytebufferpool v1.0.0 // indirect
	github.com/valyala/fasthttp v1.51.0 // indirect
	github.com/valyala/tcplisten v1.0.0 // indirect
	golang.org/x/sys v0.28.0 // indirect
)
```

#### 3. .air.toml

```toml
# Air configuration for Go hot reload
# https://github.com/air-verse/air

root = "."
testdata_dir = "testdata"
tmp_dir = "tmp"

[build]
  args_bin = []
  bin = "./tmp/main"
  cmd = "go build -o ./tmp/main ."
  delay = 1000
  exclude_dir = ["assets", "tmp", "vendor", "testdata"]
  exclude_file = []
  exclude_regex = ["_test.go"]
  exclude_unchanged = false
  follow_symlink = false
  full_bin = ""
  include_dir = []
  include_ext = ["go", "tpl", "tmpl", "html"]
  include_file = []
  kill_delay = "0s"
  log = "build-errors.log"
  poll = false
  poll_interval = 0
  post_cmd = []
  pre_cmd = []
  rerun = false
  rerun_delay = 500
  send_interrupt = false
  stop_on_error = false

[color]
  app = ""
  build = "yellow"
  main = "magenta"
  runner = "green"
  watcher = "cyan"

[log]
  main_only = false
  time = false

[misc]
  clean_on_exit = false

[screen]
  clear_on_rebuild = false
  keep_scroll = true
```

#### 4. Dockerfile

```dockerfile
# Build stage - สำหรับ development และ compile
FROM golang:1.24-alpine AS builder

# ติดตั้ง tools ที่จำเป็นสำหรับ development
RUN apk add --no-cache git

# กำหนด Working Directory ภายใน Container
WORKDIR /app

# Copy go mod files เข้าไปก่อน
# เพื่อใช้ประโยชน์จาก Docker cache layer ทำให้ไม่ต้อง download dependencies ใหม่ทุกครั้งที่แก้โค้ด
COPY go.mod go.sum ./

# Download dependencies
RUN go mod download && go mod verify

# Copy โค้ดทั้งหมดในโปรเจกต์เข้าไปใน container
COPY . .

# Build the application
# CGO_ENABLED=0: สำหรับ static binary
# GOOS=linux: target OS
# -a: force rebuilding
# -installsuffix cgo: เพิ่ม suffix เพื่อไม่ให้ปนกับ cache
# -o: output filename
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o fiber-app .

# Production stage - สำหรับ production deployment
FROM alpine:latest AS production

# ติดตั้ง ca-certificates สำหรับ HTTPS requests
RUN apk --no-cache add ca-certificates

# กำหนด Working Directory ภายใน Container
WORKDIR /root/

# Copy binary ที่ compiled แล้วจาก builder stage
COPY --from=builder /app/fiber-app .

# กำหนด Port ที่ Container จะทำงาน
EXPOSE 9000

# คำสั่งสำหรับรัน Fiber Application
CMD ["./fiber-app"]
```

#### 5. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # สร้าง image จาก Dockerfile แต่จะใช้แค่ 'builder' stage เป็นฐาน
    # เพื่อให้มี Go development tools ครบ
    build:
      context: .
      target: builder # <-- บอกให้ build ถึงแค่ stage 'builder'
    container_name: fiber-app-dev-instance
    ports:
      - "9002:9000" # Map port 9002 (host) -> 9000 (container)
    volumes:
      # เชื่อมโค้ดจากเครื่องเราเข้าไปใน container แบบ Real-time
      - ./main.go:/app/main.go:ro
      - ./go.mod:/app/go.mod:ro
      - ./go.sum:/app/go.sum:ro
      - ./.air.toml:/app/.air.toml:ro
    environment:
      - PORT=9000
      - CGO_ENABLED=0
    # รันด้วย go run เพื่อ development
    command: go run main.go
    stdin_open: true
    tty: true
```

#### 6. .dockerignore

```dockerignore
# Git
.git
.gitignore

# Docker
Dockerfile
docker-compose*.yml
.dockerignore

# Documentation
README.md
*.md

# Binary files
*.exe
main
fiber-app

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Go
*.log
*.test
*.out
vendor/
tmp/

# CI/CD
Jenkinsfile
.github/
.gitlab-ci.yml

# Air
.air.toml
build-errors.log
```

#### 7. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: สร้างฟังก์ชันสำหรับส่ง Notification ไปยัง n8n
// การสร้างฟังก์ชันช่วยลดการเขียนโค้ดซ้ำซ้อน (DRY Principle)
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    agent any

    options { 
        skipDefaultCheckout(true)
    }

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/fiber-docker-app"

        // DEV environment
        DEV_APP_NAME              = "fiber-app-dev"
        DEV_HOST_PORT             = "9001"

        // PROD environment
        PROD_APP_NAME             = "fiber-app-prod"
        PROD_HOST_PORT            = "9000"
    }

    parameters {
        choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
        string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag ที่ต้องการ (เช่น Git Hash หรือ dev-123)')
        choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือกว่าจะ Rollback ที่ Environment ไหน')
    }

    stages {

        // Stage 1: Checkout
        stage('Checkout') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        // Stage 2: Build & Test
        stage('Build & Test') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Running tests inside a consistent Docker environment..."
                script {
                    docker.image('golang:1.24-alpine').inside {
                        sh '''
                            apk add --no-cache git
                            go mod download
                            go mod verify
                            go build -v ./...
                            go test -v ./...
                        '''
                    }
                }
            }
        }

        // Stage 3: Build & Push Docker Image
        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag
                    
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "--target production .")
                        
                        echo "Pushing images to Docker Hub..."
                        customImage.push()
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        // Deploy to DEV
        stage('Deploy to DEV (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            } 
            steps {
                script {
                    def deployCmd = """
                        echo "Deploying container ${DEV_APP_NAME} from latest image..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${DEV_APP_NAME} || true
                        docker rm ${DEV_APP_NAME} || true
                        docker run -d --name ${DEV_APP_NAME} \
                            -p ${DEV_HOST_PORT}:9000 \
                            -e PORT=9000 \
                            ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${DEV_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                    """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV (Local Docker)', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        // Approval for Production
        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Deploy image tag '${env.IMAGE_TAG}' to PRODUCTION (Local Docker on port ${PROD_HOST_PORT})?"
                }
            }
        }

        // Deploy to PROD
        stage('Deploy to PRODUCTION (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            } 
            steps {
                script {
                    def deployCmd = """
                        echo "Deploying container ${PROD_APP_NAME} from latest image..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${PROD_APP_NAME} || true
                        docker rm ${PROD_APP_NAME} || true
                        docker run -d --name ${PROD_APP_NAME} \
                            -p ${PROD_HOST_PORT}:9000 \
                            -e PORT=9000 \
                            ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${PROD_APP_NAME} --format "table {{.Names}}\\t{{.Image}}\\t{{.Status}}"
                    """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION (Local Docker)', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        // Rollback
        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (params.ROLLBACK_TAG.trim().isEmpty()) {
                        error "เมื่อเลือก Rollback กรุณาระบุ 'ROLLBACK_TAG'"
                    }

                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"
                    
                    echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to image: ${imageToDeploy}"
                    
                    def deployCmd = """
                        docker pull ${imageToDeploy}
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                        docker run -d --name ${targetAppName} \
                            -p ${targetHostPort}:9000 \
                            -e PORT=9000 \
                            ${imageToDeploy}
                    """
                    sh(deployCmd)
                }
            }
            post {
                success {
                    script {
                        def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                        def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                        sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "Cleaning up Docker images on agent..."
                    try {
                        sh """
                            docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                            docker image rm -f ${DOCKER_REPO}:latest || true
                        """
                    } catch (err) {
                        echo "Could not clean up images, but continuing..."
                    }
                }
            }
        }
        failure {
            sendNotificationToN8n('failed', "Pipeline Failed", 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 8. Push to GitHub

```bash
git add .
git commit -m "Initial Fiber Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 9. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 10. แก้ไข main.go เพื่อทดสอบ

```go
package main

import (
	"log"
	"os"
	"os/signal"
	"syscall"
	"time"

	"github.com/gofiber/fiber/v2"
	"github.com/gofiber/fiber/v2/middleware/logger"
	"github.com/gofiber/fiber/v2/middleware/recover"
)

func main() {
	// Create Fiber instance with custom config
	app := fiber.New(fiber.Config{
		AppName:          "Fiber Docker App v1.0.0",
		ServerHeader:     "Fiber",
		ReadBufferSize:   16384,
		WriteBufferSize:  16384,
		BodyLimit:        4 * 1024 * 1024,
		DisableKeepalive: false,
	})

	// Middleware
	app.Use(logger.New())
	app.Use(recover.New())

	// Routes
	app.Get("/", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"message": "Hello World 🌈",
			"status":  "success",
			"app":     "Fiber Docker App",
			"version": "1.0.0",
		})
	})

	app.Get("/ping", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"message": "Pong 🏓",
			"status":  "success",
		})
	})

	app.Get("/health", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"status":  "ok",
			"uptime":  "running",
			"time":    time.Now().Format(time.RFC3339),
		})
	})

	// เพิ่ม endpoint ใหม่สำหรับทดสอบ
	app.Get("/api/info", func(c *fiber.Ctx) error {
		return c.JSON(fiber.Map{
			"app":        "Fiber Docker App",
			"version":    "1.0.0",
			"framework":  "Fiber v2",
			"go_version": "1.24.2",
			"port":       os.Getenv("PORT"),
		})
	})

	app.Get("/api/products", func(c *fiber.Ctx) error {
		products := []fiber.Map{
			{
				"id":      1,
				"name":    "Laptop",
				"price":   25000.00,
				"inStock": true,
			},
			{
				"id":      2,
				"name":    "Mouse",
				"price":   500.00,
				"inStock": true,
			},
			{
				"id":      3,
				"name":    "Keyboard",
				"price":   1200.00,
				"inStock": false,
			},
			{
				"id":      4,
				"name":    "Monitor",
				"price":   8000.00,
				"inStock": true,
			},
		}
		return c.JSON(fiber.Map{
			"status":   "success",
			"products": products,
			"count":    len(products),
		})
	})

	// Get port from environment or use default
	port := os.Getenv("PORT")
	if port == "" {
		port = "9000"
	}

	// Channel to listen for interrupt signals for graceful shutdown
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, os.Interrupt, syscall.SIGTERM)

	// Start server in a goroutine
	go func() {
		log.Printf("🚀 Server is starting on port %s...", port)
		log.Printf("🌐 Visit: http://localhost:%s", port)
		if err := app.Listen(":" + port); err != nil {
			log.Printf("❌ Error starting server: %v", err)
		}
	}()

	// Wait for interrupt signal
	<-quit
	log.Println("\n🛑 Shutting down server...")

	// Graceful shutdown
	if err := app.Shutdown(); err != nil {
		log.Printf("❌ Error during shutdown: %v", err)
	}

	log.Println("✅ Server stopped gracefully")
}
```

#### 11. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "Fiber-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 12. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 13. ทดสอบ API endpoints
```bash
# ทดสอบ DEV environment (port 9001)
curl http://localhost:9001/
curl http://localhost:9001/ping
curl http://localhost:9001/health
curl http://localhost:9001/api/info
curl http://localhost:9001/api/products

# ทดสอบ PROD environment (port 9000)
curl http://localhost:9000/
curl http://localhost:9000/ping
curl http://localhost:9000/health
curl http://localhost:9000/api/info
curl http://localhost:9000/api/products
```

#### 14. ทดสอบการทำงานของ Hot Reload (Development Mode)
```bash
# รัน docker-compose สำหรับ development
cd fiber-docker-app
docker-compose -f docker-compose.dev.yml up

# แก้ไขไฟล์ main.go
# บันทึกไฟล์ แล้ว Go จะ rebuild และ restart application อัตโนมัติ

# เปิดเบราว์เซอร์ไปที่
http://localhost:9002
```

#### 15. คำสั่งที่เป็นประโยชน์

```bash
# ดู logs ของ container
docker logs fiber-app-dev
docker logs fiber-app-prod

# เข้าไปใน container
docker exec -it fiber-app-dev sh
docker exec -it fiber-app-prod sh

# ตรวจสอบ container ที่กำลังรัน
docker ps

# หยุด container
docker stop fiber-app-dev fiber-app-prod

# ลบ container
docker rm fiber-app-dev fiber-app-prod

# ดู Docker images
docker images | grep fiber-docker-app

# ลบ Docker image
docker rmi iamsamitdev/fiber-docker-app:latest

# รัน Go commands ใน container
docker exec -it fiber-app-dev go version
docker exec -it fiber-app-dev go mod tidy

# Build แบบ local
go build -o fiber-app main.go
./fiber-app

# Run tests
go test -v ./...

# Format code
go fmt ./...
```

#### 16. ข้อมูลเพิ่มเติมเกี่ยวกับ Go Fiber

- **Fiber** เป็น Express-inspired web framework ที่สร้างบน Fasthttp
- รองรับ **Go 1.24** (เวอร์ชันล่าสุด)
- มี **Performance สูงกว่า** frameworks อื่นๆ ใน Go
- ใช้ **Zero Allocation Router** สำหรับความเร็วสูงสุด
- รองรับ **Middleware** หลากหลายแบบ
- มี **Graceful Shutdown** และ **Hot Reload** ด้วย Air
- Docker images ขนาดเล็กมาก (~10MB) ด้วย **Alpine-based images**
- Binary ที่ compile แล้วเป็น **Static Binary** ไม่ต้องพึ่งพา dependencies


### PHP Laravel Jenkins multibranch pipeline

### 🏗️ Project Structure

```
laravel-docker-app/
├── 📁 app/
│   ├── 📁 Http/
│   │   └── 📁 Controllers/
│   │       └── 📄 Controller.php          # Base controller
│   ├── 📁 Models/
│   │   └── 📄 User.php                    # User model
│   └── 📁 Providers/
│       └── 📄 AppServiceProvider.php      # Service provider
├── 📁 bootstrap/
│   ├── 📄 app.php                         # Application bootstrap
│   ├── 📄 providers.php                   # Service providers
│   └── 📁 cache/                          # Bootstrap cache
├── 📁 config/                             # Configuration files
│   ├── 📄 app.php
│   ├── 📄 database.php
│   └── 📄 ...
├── 📁 database/
│   ├── 📄 database.sqlite                 # SQLite database for testing
│   ├── 📁 factories/
│   │   └── 📄 UserFactory.php
│   ├── 📁 migrations/
│   └── 📁 seeders/
│       └── 📄 DatabaseSeeder.php
├── 📁 docker/
│   └── 📁 nginx/
│       └── 📄 default.conf                # Nginx configuration
├── 📁 public/
│   ├── 📄 index.php                       # Application entry point
│   └── 📄 ...
├── 📁 resources/
│   ├── 📁 css/
│   │   └── 📄 app.css
│   ├── 📁 js/
│   │   ├── 📄 app.js
│   │   └── 📄 bootstrap.js
│   └── 📁 views/
│       └── 📄 welcome.blade.php           # Welcome page
├── 📁 routes/
│   ├── 📄 web.php                         # Web routes
│   └── 📄 console.php                     # Console routes
├── 📁 storage/                            # Storage directory
│   ├── 📁 app/
│   ├── 📁 framework/
│   │   ├── 📁 cache/
│   │   ├── 📁 sessions/
│   │   └── 📁 views/
│   └── 📁 logs/
├── 📁 tests/
│   ├── 📄 TestCase.php
│   ├── 📁 Feature/
│   │   └── 📄 ExampleTest.php
│   └── 📁 Unit/
│       └── 📄 ExampleTest.php
├── 📁 vendor/                             # Composer dependencies
├── 📄 .dockerignore                       # Files to ignore in Docker build
├── 📄 .env.example                        # Environment variables example
├── 📄 .gitignore                          # Files to ignore in Git
├── 📄 artisan                             # Laravel CLI tool
├── 📄 composer.json                       # Composer configuration
├── 📄 composer.lock                       # Composer lock file
├── 🐳 Dockerfile                           # Docker build configuration (Multi-stage)
├── 🐳 docker-compose.dev.yml               # Docker Compose for development
├── 🔧 Jenkinsfile                          # Jenkins CI/CD pipeline
├── 📄 package.json                        # Node.js configuration
├── 📄 phpunit.xml                         # PHPUnit configuration
├── 📄 supervisord.conf                    # Supervisor configuration
├── 📄 vite.config.js                      # Vite configuration
└── 📄 README.md                           # Project documentation
```

#### 1. composer.json

```json
{
    "$schema": "https://getcomposer.org/schema.json",
    "name": "laravel/laravel",
    "type": "project",
    "description": "The skeleton application for the Laravel framework.",
    "keywords": ["laravel", "framework"],
    "license": "MIT",
    "require": {
        "php": "^8.2",
        "laravel/framework": "^12.0",
        "laravel/tinker": "^2.10.1"
    },
    "require-dev": {
        "fakerphp/faker": "^1.23",
        "laravel/pail": "^1.2.2",
        "laravel/pint": "^1.24",
        "laravel/sail": "^1.41",
        "mockery/mockery": "^1.6",
        "nunomaduro/collision": "^8.6",
        "phpunit/phpunit": "^11.5.3"
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/",
            "Database\\Factories\\": "database/factories/",
            "Database\\Seeders\\": "database/seeders/"
        }
    },
    "autoload-dev": {
        "psr-4": {
            "Tests\\": "tests/"
        }
    },
    "scripts": {
        "post-autoload-dump": [
            "Illuminate\\Foundation\\ComposerScripts::postAutoloadDump",
            "@php artisan package:discover --ansi"
        ],
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ],
        "post-root-package-install": [
            "@php -r \"file_exists('.env') || copy('.env.example', '.env');\""
        ],
        "post-create-project-cmd": [
            "@php artisan key:generate --ansi",
            "@php -r \"file_exists('database/database.sqlite') || touch('database/database.sqlite');\"",
            "@php artisan migrate --graceful --ansi"
        ]
    },
    "extra": {
        "laravel": {
            "dont-discover": []
        }
    },
    "config": {
        "optimize-autoloader": true,
        "preferred-install": "dist",
        "sort-packages": true,
        "allow-plugins": {
            "pestphp/pest-plugin": true,
            "php-http/discovery": true
        }
    },
    "minimum-stability": "stable",
    "prefer-stable": true
}
```

#### 2. routes/web.php

```php
<?php

use Illuminate\Support\Facades\Route;

Route::get('/', function () {
    return view('welcome');
});

Route::get('/api/hello', function () {
    return response()->json([
        'message' => 'Hello from Laravel API!',
        'status' => 'success',
        'framework' => 'Laravel 12',
        'php_version' => PHP_VERSION,
    ]);
});

Route::get('/api/health', function () {
    return response()->json([
        'status' => 'healthy',
        'timestamp' => now()->toIso8601String(),
        'service' => 'Laravel Docker App',
    ]);
});
```

#### 3. phpunit.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true"
>
    <testsuites>
        <testsuite name="Unit">
            <directory>tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory>tests/Feature</directory>
        </testsuite>
    </testsuites>
    <source>
        <include>
            <directory>app</directory>
        </include>
    </source>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="APP_MAINTENANCE_DRIVER" value="file"/>
        <env name="BCRYPT_ROUNDS" value="4"/>
        <env name="CACHE_STORE" value="array"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
        <env name="MAIL_MAILER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
    </php>
</phpunit>
```

#### 4. Dockerfile

```dockerfile
# ==============================
# Build Stage - For installing dependencies and compiling assets
# ==============================
FROM php:8.2-fpm-alpine AS builder

# Install system dependencies
RUN apk add --no-cache \
    git curl libpng-dev libxml2-dev libzip-dev \
    zip unzip nodejs npm oniguruma-dev

# Install PHP extensions
RUN docker-php-ext-install pdo pdo_mysql mbstring exif pcntl bcmath gd zip

# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Set working directory
WORKDIR /var/www/html

# Install dependencies WITHOUT running scripts first
COPY composer.json composer.lock ./
RUN composer install --prefer-dist --no-interaction --no-progress --no-scripts --no-autoloader

# Copy package files
COPY package*.json ./
RUN npm install

# Now, copy all application code
COPY . .

# Now that 'artisan' exists, run the scripts
RUN composer dump-autoload --optimize

# Build frontend assets
RUN npm run build

# Remove development dependencies
RUN npm prune --production

# ==============================
# Production Stage - Nginx + PHP-FPM + Supervisord
# ==============================
FROM nginx:alpine AS production

# Install Supervisor and PHP-FPM
RUN apk add --no-cache supervisor php82-fpm php82-pdo php82-pdo_mysql \
    php82-mbstring php82-xml php82-openssl php82-tokenizer php82-session \
    php82-fileinfo php82-zip php82-gd php82-dom php82-bcmath

# Copy application code from the builder stage
COPY --from=builder /var/www/html /var/www/html

# Copy Nginx config
COPY --from=builder /var/www/html/docker/nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy Supervisor config
COPY --from=builder /var/www/html/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Set Permissions
RUN chown -R nginx:nginx /var/www/html \
    && chmod -R 775 /var/www/html/storage \
    && chmod -R 775 /var/www/html/bootstrap/cache

WORKDIR /var/www/html

EXPOSE 80

# Use supervisord to run Nginx and PHP-FPM
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]
```

#### 5. docker/nginx/default.conf

```nginx
server {
    listen 80;
    server_name localhost;
    root /var/www/html/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include        fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

#### 6. supervisord.conf

```ini
[supervisord]
nodaemon=true
user=root
logfile=/var/log/supervisor/supervisord.log
pidfile=/var/run/supervisord.pid

[program:php-fpm]
command=php-fpm82 -F
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
autorestart=true
startretries=3

[program:nginx]
command=nginx -g 'daemon off;'
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
autorestart=true
startretries=3
```

#### 7. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # สร้าง image จาก Dockerfile แต่จะใช้แค่ 'builder' stage เป็นฐาน
    build:
      context: .
      target: builder
    container_name: laravel-app-dev-instance
    ports:
      - "8002:8000"
    volumes:
      # เชื่อมโค้ดจากเครื่องเราเข้าไปใน container แบบ Real-time
      - ./:/var/www/html
      # ไม่ map vendor/, node_modules/ เพื่อใช้ของใน container
      - /var/www/html/vendor
      - /var/www/html/node_modules
      - /var/www/html/storage/framework/cache
      - /var/www/html/storage/framework/sessions
      - /var/www/html/storage/framework/views
    environment:
      - APP_ENV=local
      - APP_DEBUG=true
      - DB_CONNECTION=mysql
      - DB_HOST=mysql
      - DB_PORT=3306
      - DB_DATABASE=laravel
      - DB_USERNAME=laravel
      - DB_PASSWORD=secret
    entrypoint: >
      sh -c "
        composer install --no-interaction &&
        php artisan serve --host=0.0.0.0 --port=8000
      "
    depends_on:
      - mysql
    networks:
      - laravel-network

  mysql:
    image: mysql:8.0
    container_name: laravel-mysql-dev
    ports:
      - "3307:3306"
    environment:
      - MYSQL_DATABASE=laravel
      - MYSQL_USER=laravel
      - MYSQL_PASSWORD=secret
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - laravel-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

networks:
  laravel-network:
    driver: bridge

volumes:
  mysql-data:
    driver: local
```

#### 8. .dockerignore

```dockerignore
# Git
.git
.gitignore
.gitattributes

# Docker
Dockerfile
docker-compose*.yml
.dockerignore
docker/

# Environment
.env
.env.*
!.env.example

# IDE
.vscode/
.idea/
*.swp
*.swo
*~
.DS_Store

# Laravel
/vendor
/node_modules
/public/hot
/public/storage
/storage/*.key
/storage/framework/cache/*
/storage/framework/sessions/*
/storage/framework/testing/*
/storage/framework/views/*
/storage/logs/*
bootstrap/cache/*

# Testing
.phpunit.result.cache
.phpunit.cache
/tests/

# Build artifacts
/public/build
/public/css
/public/js
/public/mix-manifest.json

# Documentation
README.md
*.md
docs/

# CI/CD
Jenkinsfile
.github/
.gitlab-ci.yml

# Logs
*.log
npm-debug.log*
yarn-debug.log*
```

#### 9. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: สร้างฟังก์ชันสำหรับส่ง Notification ไปยัง n8n
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        buildDiscarder(logRotator(numToKeepStr: '10'))
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/laravel-docker-app"

        // DEV environment
        DEV_APP_NAME              = "laravel-app-dev"
        DEV_HOST_PORT             = "8001"

        // PROD environment
        PROD_APP_NAME             = "laravel-app-prod"
        PROD_HOST_PORT            = "8000"
    }

    parameters {
        choice(
            name: 'ACTION',
            choices: ['Build & Deploy', 'Rollback'],
            description: 'เลือก Action ที่ต้องการทำ'
        )
        string(
            name: 'ROLLBACK_TAG',
            defaultValue: '',
            description: 'ระบุ Image Tag ที่ต้องการ Rollback'
        )
        choice(
            name: 'ROLLBACK_TARGET',
            choices: ['dev', 'prod'],
            description: 'เลือก Environment ที่ต้องการ Rollback'
        )
    }

    stages {

        // Stage 1: Checkout
        stage('Checkout') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "🔄 Checking out code..."
                checkout scm
            }
        }

        // Stage 2: Install & Test
        stage('Install & Test') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    echo "📦 Installing dependencies and running tests..."
                    withCredentials([string(credentialsId: 'laravel-app-key', variable: 'APP_KEY')]) {
                        docker.image('composer:latest').inside('-u root:root') {
                            sh '''
                                set -eux
                                composer install --prefer-dist --no-interaction --no-progress --optimize-autoloader
                                
                                cp .env.example .env
                                sed -i "s|APP_KEY=.*|APP_KEY=${APP_KEY}|g" .env
                                sed -i "s|DB_CONNECTION=.*|DB_CONNECTION=sqlite|g" .env
                                sed -i "s|# DB_DATABASE=|DB_DATABASE=/var/www/html/database/database.sqlite|g" .env
                                touch database/database.sqlite

                                php artisan test
                            '''
                        }
                    }
                }
            }
        }

        // Stage 3: Build & Push Docker Image
        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag

                    echo "🐳 Building and pushing Docker image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                    
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "--target production --no-cache .")
                        customImage.push()
                        
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        // Deploy to DEV
        stage('Deploy to DEV') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            }
            steps {
                script {
                    echo "🚀 Deploying to DEV environment..."
                    
                    sh """
                        docker stop ${DEV_APP_NAME} || true
                        docker rm ${DEV_APP_NAME} || true
                    """
                    
                    withCredentials([
                        file(credentialsId: 'laravel-env-dev', variable: 'ENV_FILE')
                    ]) {
                        sh """
                            docker run -d \
                                --name ${DEV_APP_NAME} \
                                -p ${DEV_HOST_PORT}:80 \
                                --env-file \${ENV_FILE} \
                                ${DOCKER_REPO}:${env.IMAGE_TAG}
                            
                            sleep 5
                            docker ps | grep ${DEV_APP_NAME}
                        """
                    }
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        // Approval for Production
        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                script {
                    echo "⏸️  Waiting for approval to deploy to PRODUCTION..."
                    
                    timeout(time: 30, unit: 'MINUTES') {
                        input message: 'Deploy to Production?',
                              ok: 'Deploy',
                              submitter: 'admin'
                    }
                }
            }
        }

        // Deploy to PROD
        stage('Deploy to PRODUCTION') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                script {
                    echo "🚀 Deploying to PRODUCTION environment..."
                    
                    sh """
                        docker stop ${PROD_APP_NAME} || true
                        docker rm ${PROD_APP_NAME} || true
                    """
                    
                    withCredentials([
                        file(credentialsId: 'laravel-env-prod', variable: 'ENV_FILE')
                    ]) {
                        sh """
                            docker run -d \
                                --name ${PROD_APP_NAME} \
                                -p ${PROD_HOST_PORT}:80 \
                                --env-file \${ENV_FILE} \
                                --restart unless-stopped \
                                ${DOCKER_REPO}:${env.IMAGE_TAG}
                            
                            sleep 5
                            docker ps | grep ${PROD_APP_NAME}
                        """
                    }
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        // Rollback
        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (!params.ROLLBACK_TAG) {
                        error("❌ ROLLBACK_TAG is required!")
                    }
                    
                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def envCredId = (params.ROLLBACK_TARGET == 'dev') ? 'laravel-env-dev' : 'laravel-env-prod'
                    
                    echo "🔄 Rolling back ${params.ROLLBACK_TARGET.toUpperCase()} to ${params.ROLLBACK_TAG}"
                    
                    sh """
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                    """
                    
                    withCredentials([file(credentialsId: envCredId, variable: 'ENV_FILE')]) {
                        sh """
                            docker run -d \
                                --name ${targetAppName} \
                                -p ${targetHostPort}:80 \
                                --env-file \${ENV_FILE} \
                                ${DOCKER_REPO}:${params.ROLLBACK_TAG}
                            
                            sleep 5
                            docker ps | grep ${targetAppName}
                        """
                    }
                }
            }
            post {
                success {
                    script {
                        def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                        def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                        sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", params.ROLLBACK_TAG, targetAppName, targetHostPort)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "🧹 Cleaning up Docker images..."
                    try {
                        sh """
                            docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                            docker image rm -f ${DOCKER_REPO}:latest || true
                        """
                    } catch (err) {
                        echo "Could not clean up images: ${err}"
                    }
                }
            }
        }
        failure {
            sendNotificationToN8n('failed', "Pipeline Failed", 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 10. Push to GitHub

```bash
git add .
git commit -m "Initial Laravel Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 11. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 12. สร้าง Jenkins Credentials สำหรับ Laravel

**1. สร้าง APP_KEY Credential:**
- ไปที่ Jenkins Dashboard > Manage Jenkins > Credentials
- คลิก "Add Credentials"
- Kind: Secret text
- Secret: `base64:your-generated-key-here` (ใช้คำสั่ง `php artisan key:generate --show`)
- ID: `laravel-app-key`
- คลิก "Create"

**2. สร้าง .env file สำหรับ DEV:**
- คลิก "Add Credentials"
- Kind: Secret file
- File: อัปโหลดไฟล์ `.env.dev`
- ID: `laravel-env-dev`
- คลิก "Create"

**3. สร้าง .env file สำหรับ PROD:**
- คลิก "Add Credentials"
- Kind: Secret file
- File: อัปโหลดไฟล์ `.env.prod`
- ID: `laravel-env-prod`
- คลิก "Create"

#### 13. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "Laravel-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 14. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 15. ทดสอบ Application
```bash
# ทดสอบ DEV environment (port 8001)
curl http://localhost:8001/
curl http://localhost:8001/api/hello
curl http://localhost:8001/api/health

# เปิดเบราว์เซอร์
http://localhost:8001

# ทดสอบ PROD environment (port 8000)
curl http://localhost:8000/
curl http://localhost:8000/api/hello
curl http://localhost:8000/api/health

# เปิดเบราว์เซอร์
http://localhost:8000
```

#### 16. ทดสอบการทำงานของ Hot Reload (Development Mode)
```bash
# รัน docker-compose สำหรับ development
cd laravel-docker-app
docker-compose -f docker-compose.dev.yml up

# แก้ไขไฟล์ routes/web.php หรือ resources/views/
# บันทึกไฟล์ แล้วรีเฟรชเบราว์เซอร์

# เปิดเบราว์เซอร์ไปที่
http://localhost:8002
```

#### 17. คำสั่งที่เป็นประโยชน์

```bash
# ดู logs ของ container
docker logs laravel-app-dev
docker logs laravel-app-prod

# เข้าไปใน container
docker exec -it laravel-app-dev sh
docker exec -it laravel-app-prod sh

# ตรวจสอบ container ที่กำลังรัน
docker ps

# หยุด container
docker stop laravel-app-dev laravel-app-prod

# ลบ container
docker rm laravel-app-dev laravel-app-prod

# ดู Docker images
docker images | grep laravel-docker-app

# ลบ Docker image
docker rmi iamsamitdev/laravel-docker-app:latest

# รัน Laravel commands ใน container
docker exec -it laravel-app-dev php artisan --version
docker exec -it laravel-app-dev php artisan migrate
docker exec -it laravel-app-dev php artisan route:list
docker exec -it laravel-app-dev php artisan tinker

# รัน Composer commands
docker exec -it laravel-app-dev composer update
docker exec -it laravel-app-dev composer require package-name

# รัน tests
docker exec -it laravel-app-dev php artisan test

# Clear cache
docker exec -it laravel-app-dev php artisan cache:clear
docker exec -it laravel-app-dev php artisan config:clear
docker exec -it laravel-app-dev php artisan route:clear
docker exec -it laravel-app-dev php artisan view:clear
```

#### 18. ข้อมูลเพิ่มเติมเกี่ยวกับ Laravel 12

- **Laravel 12** เป็นเวอร์ชันล่าสุด (ณ ปี 2025)
- รองรับ **PHP 8.2+** 
- ใช้ **Composer** สำหรับจัดการ dependencies
- มี **Artisan CLI** สำหรับช่วยในการพัฒนา
- รองรับ **PHPUnit** สำหรับการทดสอบ
- ใช้ **Blade** template engine
- รองรับ **Eloquent ORM** สำหรับ database
- มี **Migration** และ **Seeder** สำหรับจัดการ database schema
- รองรับ **Vite** สำหรับ frontend assets compilation
- **Multi-stage Docker build** สำหรับ production deployment พร้อม Nginx + PHP-FPM + Supervisord

### NestJS Jenkins multibranch pipeline

### 🏗️ Project Structure

```
nestjs-docker-app/
├── 📁 src/
│   ├── 📄 main.ts                      # NestJS application entry point
│   ├── 📄 app.module.ts                # Root module
│   ├── 📄 app.controller.ts            # Main controller
│   ├── 📄 app.controller.spec.ts       # Controller unit tests
│   └── 📄 app.service.ts               # Main service
├── 📁 test/
│   ├── 📄 app.e2e-spec.ts             # End-to-end tests
│   └── 📄 jest-e2e.json               # Jest E2E configuration
├── 📁 node_modules/                    # Node.js dependencies
├── 📁 dist/                            # Compiled JavaScript output
├── 📁 .github/
│   └── 📁 workflows/
│       └── 📄 main.yml                # GitHub Actions workflow
├── 📄 .dockerignore                    # Files to ignore in Docker build
├── 📄 .gitignore                       # Files to ignore in Git
├── 📄 .prettierrc                      # Prettier configuration
├── 🐳 Dockerfile                       # Docker build configuration (Multi-stage)
├── 🐳 docker-compose.dev.yml           # Docker Compose for development
├── 🔧 Jenkinsfile                      # Jenkins CI/CD pipeline
├── ⚙️ eslint.config.mjs               # ESLint configuration
├── ⚙️ nest-cli.json                   # Nest CLI configuration
├── 📄 package.json                    # Node.js project configuration
├── 📄 package-lock.json               # Dependency lock file
├── 📄 tsconfig.json                   # TypeScript configuration
├── 📄 tsconfig.build.json             # TypeScript build configuration
└── 📄 README.md                       # Project documentation
```

#### 1. package.json

```json
{
  "name": "nestjs-docker-app",
  "version": "0.0.1",
  "description": "",
  "author": "",
  "private": true,
  "license": "UNLICENSED",
  "scripts": {
    "build": "nest build",
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    "start": "nest start",
    "start:dev": "nest start --watch",
    "start:debug": "nest start --debug --watch",
    "start:prod": "node dist/main",
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    "test": "jest",
    "test:watch": "jest --watch",
    "test:cov": "jest --coverage",
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  "dependencies": {
    "@nestjs/common": "^11.0.1",
    "@nestjs/core": "^11.0.1",
    "@nestjs/platform-express": "^11.0.1",
    "reflect-metadata": "^0.2.2",
    "rxjs": "^7.8.1"
  },
  "devDependencies": {
    "@eslint/eslintrc": "^3.2.0",
    "@eslint/js": "^9.18.0",
    "@nestjs/cli": "^11.0.0",
    "@nestjs/schematics": "^11.0.0",
    "@nestjs/testing": "^11.0.1",
    "@types/express": "^5.0.0",
    "@types/jest": "^30.0.0",
    "@types/node": "^22.10.7",
    "@types/supertest": "^6.0.2",
    "eslint": "^9.18.0",
    "eslint-config-prettier": "^10.0.1",
    "eslint-plugin-prettier": "^5.2.2",
    "globals": "^16.0.0",
    "jest": "^30.0.0",
    "prettier": "^3.4.2",
    "source-map-support": "^0.5.21",
    "supertest": "^7.0.0",
    "ts-jest": "^29.2.5",
    "ts-loader": "^9.5.2",
    "ts-node": "^10.9.2",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^5.7.3"
  },
  "jest": {
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    "rootDir": "src",
    "testRegex": ".*\\.spec\\.ts$",
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    "coverageDirectory": "../coverage",
    "testEnvironment": "node"
  }
}
```

#### 2. src/main.ts

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  // อ่านค่า PORT จาก environment variable, ถ้าไม่มีให้ใช้ 3000
  await app.listen(process.env.PORT || 3000);
}
bootstrap();
```

#### 3. src/app.controller.ts

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  @Get('health')
  getHealth(): string {
    return this.appService.getHealth();
  }
}
```

#### 4. src/app.service.ts

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello from NestJS Docker App!';
  }

  getHealth(): string {
    return 'OK';
  }
}
```

#### 5. Dockerfile

```dockerfile
# Build stage - สำหรับ development และ testing
FROM node:22-alpine AS builder

# กำหนด Working Directory ภายใน Container
WORKDIR /app

# Copy ไฟล์ package.json และ package-lock.json
COPY package*.json ./

# ติดตั้ง Dependencies
RUN npm install

# Copy โค้ดทั้งหมดในโปรเจกต์
COPY . .

# ล้าง dist เก่าทิ้งก่อน build
RUN rm -rf dist

# Build NestJS Application
RUN npm run build

# Production stage - สำหรับ production deployment
FROM node:22-alpine AS production

# กำหนด Working Directory ภายใน Container
WORKDIR /app

# Copy package files
COPY package*.json ./

# ติดตั้งเฉพาะ production dependencies
RUN npm ci --only=production && npm cache clean --force

# Copy โค้ดที่ compiled แล้วจาก builder stage
COPY --from=builder /app/dist ./dist

# กำหนด Port ที่ Container จะทำงาน
EXPOSE 3000

# Set environment variable
ENV NODE_ENV=production

# คำสั่งสำหรับรัน NestJS Application (ใช้ compiled JavaScript)
CMD ["node", "dist/main"]
```

#### 6. docker-compose.dev.yml

```yaml
services:
  app-dev:
    # สร้าง image จาก Dockerfile แต่จะใช้แค่ 'builder' stage เป็นฐาน
    # เพื่อให้มี devDependencies ครบสำหรับ development
    build:
      context: .
      target: builder # <-- บอกให้ build ถึงแค่ stage 'builder'
    container_name: nestjs-app-dev-instance
    ports:
      - "7002:3000" # Map port 7002 (host) -> 3000 (container)
    volumes:
      # Mount โฟลเดอร์ src เข้าไปเพื่อ live reload
      - ./src:/app/src
    environment:
      - NODE_ENV=development
      - PORT=3000
    # รัน NestJS ในโหมด watch สำหรับ hot reload
    command: npm run start:dev
```

#### 7. .dockerignore

```dockerignore
# Dependencies
node_modules
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Build outputs
dist
build

# Environment files
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Testing
coverage
*.lcov
.nyc_output

# Git
.git
.gitignore

# Docker
Dockerfile
.dockerignore
docker-compose*.yml

# Documentation
README.md
*.md

# IDE
.vscode
.idea
*.swp
*.swo
.DS_Store

# Logs
logs
*.log

# Temporary files
.tmp
.temp

# NestJS specific
.nest-cli.json
nest-cli.json

# Test files
test/
*.spec.ts
*.test.ts

# CI/CD
.github/
Jenkinsfile
```

#### 8. Jenkinsfile

```groovy
// =================================================================
// HELPER FUNCTION: สร้างฟังก์ชันสำหรับส่ง Notification ไปยัง n8n
// การสร้างฟังก์ชันช่วยลดการเขียนโค้ดซ้ำซ้อน (DRY Principle)
// =================================================================

def sendNotificationToN8n(String status, String stageName, String imageTag, String containerName, String hostPort) {
    // ใช้ Jenkins HTTP Request Plugin (ต้องติดตั้งก่อน)
    script {
        withCredentials([string(credentialsId: 'n8n-webhook', variable: 'N8N_WEBHOOK_URL')]) {
            def payload = [
                project  : env.JOB_NAME,
                stage    : stageName,
                status   : status,
                build    : env.BUILD_NUMBER,
                image    : "${env.DOCKER_REPO}:${imageTag}",
                container: containerName,
                url      : "http://localhost:${hostPort}/",
                timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ssXXX")
            ]
            def body = groovy.json.JsonOutput.toJson(payload)
            try {
                httpRequest acceptType: 'APPLICATION_JSON',
                            contentType: 'APPLICATION_JSON',
                            httpMode: 'POST',
                            requestBody: body,
                            url: N8N_WEBHOOK_URL,
                            validResponseCodes: '200:299'
                echo "n8n webhook (${status}) sent successfully."
            } catch (err) {
                echo "Failed to send n8n webhook (${status}): ${err}"
            }
        }
    }
}

pipeline {
    agent any

    options { 
        skipDefaultCheckout(true)
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-cred'
        DOCKER_REPO               = "iamsamitdev/nestjs-docker-app"
        
        DEV_APP_NAME              = "nestjs-app-dev"
        DEV_HOST_PORT             = "7001"
        
        PROD_APP_NAME             = "nestjs-app-prod"
        PROD_HOST_PORT            = "7000"
    }

    parameters {
        choice(name: 'ACTION', choices: ['Build & Deploy', 'Rollback'], description: 'เลือก Action ที่ต้องการ')
        string(name: 'ROLLBACK_TAG', defaultValue: '', description: 'สำหรับ Rollback: ใส่ Image Tag ที่ต้องการ')
        choice(name: 'ROLLBACK_TARGET', choices: ['dev', 'prod'], description: 'สำหรับ Rollback: เลือก Environment')
    }

    stages {

        stage('Checkout') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Install & Test') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                echo "Running tests inside a consistent Docker environment..."
                script {
                    docker.image('node:22-alpine').inside {
                        sh '''
                            if [ -f package-lock.json ]; then npm ci; else npm install; fi
                            npm test
                        '''
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            when { expression { params.ACTION == 'Build & Deploy' } }
            steps {
                script {
                    def imageTag = (env.BRANCH_NAME == 'main') ? 
                        sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim() : 
                        "dev-${env.BUILD_NUMBER}"
                    env.IMAGE_TAG = imageTag
                    
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_HUB_CREDENTIALS_ID) {
                        echo "Building image: ${DOCKER_REPO}:${env.IMAGE_TAG}"
                        def customImage = docker.build("${DOCKER_REPO}:${env.IMAGE_TAG}", "--target production .")
                        
                        echo "Pushing images to Docker Hub..."
                        customImage.push()
                        if (env.BRANCH_NAME == 'main') {
                            customImage.push('latest')
                        }
                    }
                }
            }
        }

        stage('Deploy to DEV (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'develop'
            } 
            steps {
                script {
                    def deployCmd = """
                        echo "Deploying container ${DEV_APP_NAME}..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${DEV_APP_NAME} || true
                        docker rm ${DEV_APP_NAME} || true
                        docker run -d --name ${DEV_APP_NAME} -p ${DEV_HOST_PORT}:3000 ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${DEV_APP_NAME}
                    """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to DEV', env.IMAGE_TAG, env.DEV_APP_NAME, env.DEV_HOST_PORT)
                }
            }
        }

        stage('Approval for Production') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            }
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Deploy '${env.IMAGE_TAG}' to PRODUCTION?"
                }
            }
        }

        stage('Deploy to PRODUCTION (Local Docker)') {
            when {
                expression { params.ACTION == 'Build & Deploy' }
                branch 'main'
            } 
            steps {
                script {
                    def deployCmd = """
                        echo "Deploying container ${PROD_APP_NAME}..."
                        docker pull ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker stop ${PROD_APP_NAME} || true
                        docker rm ${PROD_APP_NAME} || true
                        docker run -d --name ${PROD_APP_NAME} -p ${PROD_HOST_PORT}:3000 ${DOCKER_REPO}:${env.IMAGE_TAG}
                        docker ps --filter name=${PROD_APP_NAME}
                    """
                    sh deployCmd
                }
            }
            post {
                success {
                    sendNotificationToN8n('success', 'Deploy to PRODUCTION', env.IMAGE_TAG, env.PROD_APP_NAME, env.PROD_HOST_PORT)
                }
            }
        }

        stage('Execute Rollback') {
            when { expression { params.ACTION == 'Rollback' } }
            steps {
                script {
                    if (params.ROLLBACK_TAG.trim().isEmpty()) {
                        error "กรุณาระบุ 'ROLLBACK_TAG'"
                    }

                    def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                    def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                    def imageToDeploy = "${DOCKER_REPO}:${params.ROLLBACK_TAG.trim()}"
                    
                    echo "ROLLING BACK ${params.ROLLBACK_TARGET.toUpperCase()} to: ${imageToDeploy}"
                    
                    def deployCmd = """
                        docker pull ${imageToDeploy}
                        docker stop ${targetAppName} || true
                        docker rm ${targetAppName} || true
                        docker run -d --name ${targetAppName} -p ${targetHostPort}:3000 ${imageToDeploy}
                    """
                    sh(deployCmd)
                }
            }
            post {
                success {
                    script {
                        def targetAppName = (params.ROLLBACK_TARGET == 'dev') ? DEV_APP_NAME : PROD_APP_NAME
                        def targetHostPort = (params.ROLLBACK_TARGET == 'dev') ? DEV_HOST_PORT : PROD_HOST_PORT
                        sendNotificationToN8n('success', "Rollback ${params.ROLLBACK_TARGET.toUpperCase()}", 
                            params.ROLLBACK_TAG, targetAppName, targetHostPort)
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                if (params.ACTION == 'Build & Deploy') {
                    echo "Cleaning up Docker images..."
                    try {
                        sh """
                            docker image rm -f ${DOCKER_REPO}:${env.IMAGE_TAG} || true
                            docker image rm -f ${DOCKER_REPO}:latest || true
                        """
                    } catch (err) {
                        echo "Could not clean up images: ${err}"
                    }
                }
                echo "Cleaning up workspace..."
                cleanWs()
            }
        }
        failure {
            sendNotificationToN8n('failed', "Pipeline Failed", 'N/A', 'N/A', 'N/A')
        }
    }
}
```

#### 9. Push to GitHub

```bash
git add .
git commit -m "Initial NestJS Docker App with Jenkins Pipeline"
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

#### 10. แยก Branch สำหรับ DEV และ PROD

```bash
git checkout -b develop
git push origin develop
```

#### 11. แก้ไข app.service.ts เพื่อทดสอบ

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello from NestJS Docker App!';
  }

  getHealth(): string {
    return 'OK';
  }

  // เพิ่ม method ใหม่สำหรับทดสอบ
  getInfo(): object {
    return {
      app: 'NestJS Docker App',
      version: '1.0.0',
      framework: 'NestJS 11',
      node: process.version,
      timestamp: new Date().toISOString()
    };
  }
}
```

#### 12. เพิ่ม endpoint ใหม่ใน app.controller.ts

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  @Get('health')
  getHealth(): string {
    return this.appService.getHealth();
  }

  @Get('info')
  getInfo(): object {
    return this.appService.getInfo();
  }
}
```

#### 13. สร้าง Jenkins Multibranch Pipeline Job ใหม่
- เปิด Jenkins Dashboard
- คลิก "New Item"
- ตั้งชื่อ Job เช่น "NestJS-Docker-App-Multibranch"
- เลือก "Multibranch Pipeline" แล้วคลิก "OK"
- ในส่วน "Branch Sources" คลิก "Add Source" แล้วเลือก "Github"
- กรอก Repository URL และเลือก Credentials ที่ตั้งค่าไว้
- ในส่วน Behavior เลือก ตามภาพ

![Jenkins Multibranch Pipeline Config](https://www.itgenius.co.th/assets/frondend/images/course_detail/devopsjenkins/itgn-1186.jpg)

- **Discover branches:** Exclude branches that are also filed as PRs
- **Discover pull requests from origin:** The current pull request revision
- **Discover pull requests from forks:** The current pull request revision
- กำหนด "strategy" เป็น The current pull request revision 
- กำหนด "Trust" เป็น From users with Admin or Write permission
- กำหนด "Property strategy" เป็น All branches get the same properties
- ในส่วน "Build Configuration" เลือก "by Jenkinsfile"
- คลิก "Save" เพื่อบันทึกการตั้งค่า

#### 14. ทดสอบการทำงานของ Jenkins Multibranch Pipeline
- Push โค้ดไปที่ branch develop เพื่อทดสอบการ deploy ไปยัง DEV
- Push โค้ดไปที่ branch main เพื่อทดสอบการ deploy ไปยัง PROD
- ทดสอบการ Rollback โดยการเลือก Action เป็น Rollback และระบุ ROLLBACK_TAG กับ ROLLBACK_TARGET
- ตรวจสอบผลลัพธ์และสถานะของ Jenkins Job

#### 15. ทดสอบ API endpoints
```bash
# ทดสอบ DEV environment (port 7001)
curl http://localhost:7001/
curl http://localhost:7001/health
curl http://localhost:7001/info

# ทดสอบ PROD environment (port 7000)
curl http://localhost:7000/
curl http://localhost:7000/health
curl http://localhost:7000/info
```

#### 16. ทดสอบการทำงานของ Hot Reload (Development Mode)
```bash
# รัน docker-compose สำหรับ development
cd nestjs-docker-app
docker-compose -f docker-compose.dev.yml up

# แก้ไขไฟล์ src/app.service.ts
# บันทึกไฟล์ แล้ว NestJS จะ rebuild และ restart application อัตโนมัติ

# เปิดเบราว์เซอร์ไปที่
http://localhost:7002
```

#### 17. คำสั่งที่เป็นประโยชน์

```bash
# ดู logs ของ container
docker logs nestjs-app-dev
docker logs nestjs-app-prod

# เข้าไปใน container
docker exec -it nestjs-app-dev sh
docker exec -it nestjs-app-prod sh

# ตรวจสอบ container ที่กำลังรัน
docker ps

# หยุด container
docker stop nestjs-app-dev nestjs-app-prod

# ลบ container
docker rm nestjs-app-dev nestjs-app-prod

# ดู Docker images
docker images | grep nestjs-docker-app

# ลบ Docker image
docker rmi iamsamitdev/nestjs-docker-app:latest

# รัน NestJS commands ใน container
docker exec -it nestjs-app-dev npm run start:dev
docker exec -it nestjs-app-dev npm test
docker exec -it nestjs-app-dev npm run test:e2e

# Build แบบ local
npm run build
npm run start:prod

# Run tests
npm test
npm run test:e2e
npm run test:cov

# Lint & Format
npm run lint
npm run format
```

#### 18. ข้อมูลเพิ่มเติมเกี่ยวกับ NestJS

- **NestJS 11** เป็นเวอร์ชันล่าสุด (ณ ปี 2025)
- รองรับ **Node.js 22+** และ **TypeScript 5.7+**
- ใช้ **Express** เป็น HTTP server เริ่มต้น (หรือเปลี่ยนเป็น Fastify ได้)
- Architecture แบบ **Modular** และ **Dependency Injection**
- รองรับ **Decorators** สำหรับ routing และ metadata
- มี **CLI** สำหรับสร้างโครงสร้างโปรเจกต์และ components
- รองรับ **Jest** สำหรับการทดสอบ (Unit & E2E tests)
- ใช้ **TypeScript** เป็นหลัก (compile เป็น JavaScript)
- รองรับ **Hot Reload** ในโหมด development
- **Multi-stage Docker build** ลดขนาด production image
- Docker images ขนาดเล็กด้วย **Alpine-based Node.js**
- Built-in support สำหรับ **WebSocket**, **GraphQL**, **Microservices**