# Jenkins Pipeline with Docker and Ansible

מדריך מקיף להבנת הפרויקט ורכיביו השונים.

## תוכן עניינים
1. [סקירה כללית](#סקירה-כללית)
2. [ארכיטקטורה](#ארכיטקטורה)
3. [רכיבי המערכת](#רכיבי-המערכת)
4. [הסבר על כל קובץ](#הסבר-על-כל-קובץ)
5. [תהליך ה-CI/CD](#תהליך-ה-cicd)
6. [הגדרות ואבטחה](#הגדרות-ואבטחה)
7. [פתרון בעיות נפוצות](#פתרון-בעיות-נפוצות)

## סקירה כללית
הפרויקט מממש פייפליין CI/CD מלא המשלב:
- Jenkins לאוטומציה
- Docker לקונטיינריזציה
- Ansible לדיפלוי
- שני שירותי Flask שונים

## ארכיטקטורה
המערכת מורכבת מ:
1. **Jenkins Server**: מריץ את הפייפליין
2. **Docker Hub**: מאחסן את תמונות הדוקר
3. **EC2 Instance**: שרת היעד להרצת השירותים
4. **Git Repository**: מאחסן את הקוד

## רכיבי המערכת

### 1. Jenkins Pipeline
- מוגדר ב-`Jenkinsfile`
- תומך בפרמטרים:
  - SERVICE: בחירה בין service1 ו-service2
  - TAG: תג לתמונת הדוקר
- שלבים:
  1. Checkout: משיכת קוד מ-GitHub
  2. Build: בניית תמונת דוקר
  3. Login: התחברות ל-DockerHub
  4. Push: דחיפת התמונה ל-DockerHub
  5. Deploy: הרצה באמצעות Ansible

### 2. Docker Services
#### Service1 (פורט 5000):
- אפליקציית Flask בסיסית
- נגישה דרך: http://3.72.39.83:5000

#### Service2 (פורט 5001):
- אפליקציית Flask שנייה עם אנדפוינטים נוספים
- נגישה דרך: http://3.72.39.83:5001

### 3. Ansible Deployment
- משתמש ב-inventory file להגדרת שרתי היעד
- Playbook מבצע:
  1. ניקוי קונטיינרים ישנים
  2. משיכת תמונה חדשה
  3. הרצת קונטיינר חדש

## הסבר על כל קובץ

### Jenkinsfile
\`\`\`groovy
pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "doronsun/myflaskapp"
    }
    parameters {
        choice(name: 'SERVICE', choices: ['service1', 'service2'])
        string(name: 'TAG', defaultValue: 'latest')
    }
    stages {
        // ... שלבי הפייפליין
    }
}
\`\`\`
- **agent any**: מאפשר הרצה על כל worker
- **environment**: מגדיר משתני סביבה
- **parameters**: מגדיר פרמטרים לבילד
- **stages**: מגדיר את שלבי הפייפליין

### deploy-playbook.yml
\`\`\`yaml
- name: Deploy Dockerized Flask App
  hosts: web
  become: yes
  vars:
    service_name: service1
    docker_image: doronsun/myflaskapp-service1:latest
    container_port: 5000
  tasks:
    # ... משימות הדיפלוי
\`\`\`
- **hosts**: מגדיר את שרתי היעד
- **become**: מריץ עם הרשאות sudo
- **vars**: משתנים שניתנים לדריסה
- **tasks**: הפעולות לביצוע

### inventory
\`\`\`ini
[web]
3.72.39.83 ansible_user=ubuntu ansible_ssh_private_key_file=/var/lib/jenkins/.ssh/doron-key.pem ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
\`\`\`
- מגדיר את שרתי היעד
- מגדיר פרטי התחברות SSH
- מבטל בדיקת host key

## תהליך ה-CI/CD
1. **Continuous Integration**:
   - משיכת קוד מ-GitHub
   - בניית תמונת דוקר
   - בדיקות (אם יש)

2. **Continuous Delivery**:
   - דחיפה ל-DockerHub
   - דיפלוי אוטומטי לשרת

## הגדרות ואבטחה

### הגדרות Jenkins
1. **Credentials**:
   - dockerhub-creds: לחיבור ל-DockerHub
   - SSH key: לחיבור לשרת EC2

### אבטחת EC2
1. **Security Group**:
   - פורט 22 (SSH)
   - פורט 5000 (Service1)
   - פורט 5001 (Service2)

### Docker Security
1. **Image Tagging**:
   - תיוג ייחודי לכל שירות
   - שימוש בתגיות גרסה

## פתרון בעיות נפוצות

### 1. בעיות SSH
```bash
# בדיקת הרשאות על מפתח
ls -la /var/lib/jenkins/.ssh/doron-key.pem
# צריך להיות 600
chmod 600 /var/lib/jenkins/.ssh/doron-key.pem
```

### 2. בעיות Docker
```bash
# בדיקת קונטיינרים רצים
docker ps
# הסרת קונטיינר תקוע
docker rm -f container_name
```

### 3. בעיות Port Binding
```bash
# בדיקת פורטים תפוסים
netstat -tulpn | grep LISTEN
# שחרור פורט
docker rm -f $(docker ps -q --filter "publish=5000")
```

## טיפים ללמידה
1. **הבנת Jenkins Pipeline**:
   - למד על Declarative vs Scripted Pipeline
   - הבן את מבנה ה-stages
   - תרגל שימוש ב-parameters

2. **Docker**:
   - הבן את מבנה ה-Dockerfile
   - תרגל בניית תמונות
   - למד על multi-stage builds

3. **Ansible**:
   - הבן את מבנה ה-playbook
   - תרגל כתיבת tasks
   - למד על variables והעברת פרמטרים

4. **אינטגרציה**:
   - תרגל שילוב בין הכלים
   - בנה פייפליינים משלך
   - נסה להוסיף שלבים חדשים 