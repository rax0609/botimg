當您設定 FIRST Machine Learning Toolchain（FMLTC）時，這些步驟看起來可能頗為繁瑣。以下是簡化後的步驟，幫助您更快速地完成設定：

## 簡化的設定步驟

### Google Cloud 初始設置

1. **安裝 Google Cloud SDK**
   - 按照[官方指南](https://cloud.google.com/sdk/install)安裝。

2. **創建 Google Cloud 專案**
   - 進入 [Google Cloud Console](https://console.cloud.google.com/home/dashboard)，創建並命名專案，記下專案 ID。

3. **啟用計費**

4. **設置環境變量**
    ```sh
    export FMLTC_GCLOUD_PROJECT_ID=<project id>
    gcloud config set project ${FMLTC_GCLOUD_PROJECT_ID}
    ```

5. **啟用所需 API**
    ```sh
    gcloud services enable cloudfunctions.googleapis.com
    gcloud services enable ml.googleapis.com
    gcloud services enable secretmanager.googleapis.com
    gcloud services enable compute.googleapis.com
    gcloud services enable cloudbuild.googleapis.com
    ```

6. **創建服務帳號並生成 key.json**
    ```sh
    gcloud iam service-accounts create ${FMLTC_GCLOUD_PROJECT_ID}-service-account
    gcloud projects add-iam-policy-binding ${FMLTC_GCLOUD_PROJECT_ID} --member "serviceAccount:${FMLTC_GCLOUD_PROJECT_ID}-service-account@${FMLTC_GCLOUD_PROJECT_ID}.iam.gserviceaccount.com" --role "roles/owner"
    gcloud iam service-accounts keys create key.json --iam-account ${FMLTC_GCLOUD_PROJECT_ID}-service-account@${FMLTC_GCLOUD_PROJECT_ID}.iam.gserviceaccount.com
    gcloud secrets create key_json --replication-policy="automatic" --data-file="key.json"
    ```

7. **設置 Flask 密鑰**
    ```sh
    echo "<YOUR-FLASK-SECRET-KEY>" >flask_secret_key.txt
    gcloud secrets create flask_secret_key --replication-policy="automatic" --data-file=flask_secret_key.txt
    rm flask_secret_key.txt
    ```

8. **配置 App Engine 預設服務帳號**
   - 在 [IAM 管理](https://console.cloud.google.com/iam-admin/iam?project=my_project_id)中，找到 `App Engine default service account`，編輯並添加 `Secret Manager Secret Accessor` 角色。

9. **創建 Cloud Storage 存儲桶**
    ```sh
    gsutil mb -c standard gs://${FMLTC_GCLOUD_PROJECT_ID}
    gsutil defacl set public-read gs://${FMLTC_GCLOUD_PROJECT_ID}
    gsutil mb -c standard gs://${FMLTC_GCLOUD_PROJECT_ID}-blobs
    gsutil mb -c standard gs://${FMLTC_GCLOUD_PROJECT_ID}-action-parameters
    ```

10. **創建 Datastore**
    - 進入 [Datastore](https://console.cloud.google.com/datastore/welcome?project=my_project_id)，選擇 `NATIVE MODE` 並創建資料庫。

11. **授權 TPU 服務帳號**
    ```sh
    curl -H "Authorization: Bearer $(gcloud auth print-access-token)" https://ml.googleapis.com/v1/projects/${FMLTC_GCLOUD_PROJECT_ID}:getConfig
    export FMLTC_TPU_SERVICE_ACCOUNT=<tpu service account>
    gcloud projects add-iam-policy-binding ${FMLTC_GCLOUD_PROJECT_ID}  --member serviceAccount:${FMLTC_TPU_SERVICE_ACCOUNT} --role roles/ml.serviceAgent
    ```

### 團隊信息設置

1. **創建 `teams` 文件**
    - 格式：`<program>,<team number>,<team code>`
    - 上傳到 `my_project_id-blobs/team_info/` 資料夾中。

### 克隆 fmltc 庫

1. **克隆並進入專案目錄**
    ```sh
    git clone https://github.com/FIRST-Tech-Challenge/fmltc
    cd fmltc
    ```

### 安裝依賴工具

1. **Google Closure Compiler**
    ```sh
    mkdir -p ~/tmp_fmltc/
    curl -o ~/tmp_fmltc/compiler-20200406.zip https://dl.google.com/closure-compiler/compiler-20200406.zip
    mkdir ../closure-compiler
    pushd ../closure-compiler
    unzip ~/tmp_fmltc/compiler-20200406.zip
    popd
    ```

2. **Google Closure Library**
    ```sh
    mkdir -p ~/tmp_fmltc/
    curl -o ~/tmp_fmltc/closure-library_v20200406.zip https://codeload.github.com/google/closure-library/zip/refs/tags/v20200406
    mkdir ../closure-library
    pushd ../closure-library
    unzip ~/tmp_fmltc/closure-library_v20200406.zip
    popd
    ```

3. **安裝 JDK**
   - 參考 [Oracle 官方下載頁面](https://www.oracle.com/java/technologies/downloads/)。

4. **安裝並驗證 Docker**
    ```sh
    gcloud auth configure-docker
    ```

### 設置環境變量及版本

1. **填寫 `env_variables.yaml` 中的值**
   - 替換 `<YOUR-PROJECT-ID>` 為您的專案 ID。
   - 替換 `<YOUR-ORIGIN>` 為服務的基礎 URL。

2. **設置版本**
    ```sh
    echo "{ \"version\": \"$(git rev-parse --short HEAD)\" }" > server/app_engine/app.properties
    ```

### 部署

1. **設置環境**
    ```sh
    source env_setup.sh
    ```

2. **部署 Datastore 索引**
    ```sh
    scripts/deploy_indexes.sh
    ```

3. **部署靜態內容**
    ```sh
    scripts/deploy_static.sh
    ```

4. **部署 JavaScript 代碼**
    ```sh
    scripts/deploy_js.sh
    ```

5. **部署 Cloud Function**
    ```sh
    scripts/deploy_cloud_function.sh
    ```

6. **部署 App Engine 代碼**
    ```sh
    scripts/deploy_gae.sh
    ```

7. **部署物件檢測 Docker 映像**
    ```sh
    scripts/deploy_docker_image.sh
    ```

### 測試

前往您在 Google Cloud Console 中找到的 URL，開始測試您的應用。

這些步驟應該幫助您更快速地完成 FMLTC 的設置和部署。
