Подготавливаем среду для сборки:


export DOCKER_CLI_EXPERIMENTAL=enabled
docker buildx version
docker run --rm --privileged docker/binfmt:a7996909642ee92942dcd6cff44b9b95f08dad64
cat /proc/sys/fs/binfmt_misc/qemu-aarch64    -> в выводе должно быть enabled
docker buildx create --name armbuilder 
docker buildx ls -> в выводе должен быть armbuilder
docker buildx use armbuilder
docker buildx inspect --bootstrap

----------------------------------------------------------------------------------

Собираем базовый образ:
mkdir /opt/arm64 && cd /opt/arm64 
git clone https://github.com/aerokube/images
cd images/selenium/base
Кладем в эту папку файл Dockerfile.base
собираем образ (в команде надо указать свой аккаунт на hub.docker.com и имя базового образа)
docker buildx build --progress=plain  --platform linux/arm64 --file ./Dockerfile.base --tag ${hub.docker.com account}/${image name and version} -o type=docker .
например:
docker buildx build --progress=plain  --platform linux/arm64 --file ./Dockerfile.base --tag dumbdumbych/selenium_base:u2010-en_ru_utf8.a -o type=docker . (точка в конце команды нужна)
Загружаем образ на докерхаб:
docker login
docker push ${hub.docker.com account}/${image name and version}

У меня базовый образ собирается долго, до 20 минут.

----------------------------------------------------------------------------------

Собираем образ с хромом:
cd /opt/arm64/images/static/chrome
Сначала надо скачать chromedriver. Я не уверен что это обязательно, но наверное лучше чтобы версия хромиума и chromedriver совпадали. Поэтому заглядываем на страничку репозитория с хромиумом:
https://launchpad.net/~saiarcot895/+archive/ubuntu/chromium-beta/+packages
и смотрим текущую версию chromium-browser. На данный момент она 91.

Линки на разные версии хромдрайвера доступны здесь:  https://stackoverflow.com/questions/38732822/compile-chromedriver-on-arm
Скачиваем хромдрайвер 91-й версии: wget https://github.com/electron/electron/releases/download/v13.0.1/chromedriver-v13.0.1-linux-arm64.zip
Распаковываем из архива chromedriver:  
unzip ./chromedriver-v13.0.1-linux-arm64.zip chromedriver
Кладем в эту же папку (/opt/arm64/images/static/chrome) файлы chromium.pref,  saiarcot895-ubuntu-chromium-beta-groovy.list и Dockerfile.chrome
В Dockerfile.chrome в 12-й строке указывается имя базового образа, который мы только что собрали. Туда нужно подставить свои данные ${hub.docker.com account}/${image name and version} из сборки базового образа
Запускаем сборку:
docker buildx build --progress=plain --platform linux/arm64 --file ./Dockerfile.chrome --tag ${hub.docker.com account}/selenium_vnc_chrome_arm64:91.0.a -o type=docker .



