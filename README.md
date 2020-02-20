## patchをあててseleniumをビルド

以下対応が3.141.59に入ってないためCloud Run上でエラーになるためパッチをあてて自前ビルドを行う

https://github.com/SeleniumHQ/selenium/issues/7354


```
cd selenium
git checkout selenium-3.141.59
git apppy ../patch.diff
./go selenium-server-standalone
mv buck-out/gen/java/server/src/org/openqa/grid/selenium/selenium.jar ../
cd ../
```


## GCR にimageをデプロイ

```
export PROJECT_NAME=example-app-44a1e
export BROWSER=firefox

docker pull selenium/standalone-$BROWSER:3.141.59-zirconium
docker build . -f $BROWSER/Dockerfile -t gcr.io/$PROJECT_NAME/selenium-standalone-$BROWSER:3.141.59patch-zirconium
docker push gcr.io/$PROJECT_NAME/selenium-standalone-$BROWSER:3.141.59patch-zirconium

```

## Cloud Run

```
gcloud run deploy selenium-standalone-$BROWSER --image gcr.io/$PROJECT_NAME/selenium-standalone-$BROWSER:3.141.59patch-zirconium \
--platform managed \
--allow-unauthenticated \
--memory 2Gi \
--port 4444 \
--set-env-vars "\
START_XVFB=false"

```

### めも

* Chromeでは `--no-sandbox` `--headless` `--disable-dev-shm-usage` をつけて利用する必要がある
* Firefoxでは `--disable-dev-shm-usage` 相当の設定がわかっていないのでまだ動かせない
