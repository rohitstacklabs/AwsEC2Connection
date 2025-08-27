# 0.1 Prompt simple (current session)
export PS1='\w [\u@\h] $ '

# 0.2 Prompt हमेशा simple रहे (Git Bash ~/.bashrc)
echo "export PS1='\\w [\\u@\\h] $ '" >> ~/.bashrc

# 0.3 Git related folders हटाओ (ध्यान से: इससे repo history हट जाएगी)
rm -rf .git .github

# प्रोजेक्ट root पर
# अगर wrapper है:
./mvnw clean package -DskipTests
# या
mvn clean package -DskipTests

# ==== EDIT THESE ====
KEY="EC2-connection-01.pem"   # या "connectionec2.pem"
HOST="ec2-13-126-129-210.ap-south-1.compute.amazonaws.com"  # अपनी instance का Public DNS/IP
JAR="target/EC2AwsConnection-0.0.1-SNAPSHOT.jar"            # बना हुआ JAR path
REMOTE_DIR="/home/ec2-user/app"                             # remote folder
USER="ec2-user"
# ====================

# key permission ठीक करो (Windows Git Bash में भी चलेगा)
chmod 400 "$KEY"

# EC2 पर app folder बना दो (एक बार)
ssh -i "$KEY" $USER@"$HOST" "mkdir -p '$REMOTE_DIR'"
scp -i "$KEY" "$JAR" $USER@"$HOST":"$REMOTE_DIR"/

ssh -i "$KEY" $USER@"$HOST"
sudo dnf update -y
sudo dnf install java-21-amazon-corretto -y
java -version

cd "$REMOTE_DIR"
# JAR का नाम अपने actual नाम से मैच करें
java -jar EC2AwsConnection-0.0.1-SNAPSHOT.jar


# Log live देखो
tail -f app.log

# 8080 कौन use कर रहा?
sudo lsof -i :8080

# Process ढूंढो
ps -ef | grep java

# ज़बरदस्ती kill (PID बदलें)
kill -9 29043

ssh -i "EC2-connection-01.pem" ec2-user@ec2-13-126-129-210.ap-south-1.compute.amazonaws.com
# अंदर जाकर:
sudo dnf install java-21-amazon-corretto -y
cd /home/ec2-user
mkdir -p app && cd app
nohup java -jar EC2AwsConnection-0.0.1-SNAPSHOT.jar > app.log 2>&1 &


scp -i "EC2-connection-01.pem" /d/AWS-02/Ec2ConnectionSpring-0.0.1-SNAPSHOT.jar ec2-user@ec2-13-126-129-210.ap-south-1.compute.amazonaws.com:/home/ec2-user/app/

cd /home/ec2-user/app
nohup java -jar Ec2ConnectionSpring-0.0.1-SNAPSHOT.jar > app.log 2>&1 &


# Local
cp target/EC2AwsConnection-0.0.1-SNAPSHOT.jar target/app.jar
scp -i "$KEY" target/app.jar $USER@"$HOST":"$REMOTE_DIR"/

# EC2
cd "$REMOTE_DIR"
nohup java -jar app.jar > app.log 2>&1 &
