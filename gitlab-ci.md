這邊目前先記錄疑問和感觸的筆記。

## 在gitlab-ci.yml中的管線模型是有預設狀態的：
> from https://docs.gitlab.com/ee/ci/yaml/#stages
+ `stage`預設為test，沒寫`stage`的JOB默認都是test。可以透過下面定義的JOB來測試pipeline
    ```yml
    null_stage_job:
    script:
    - echo "這個沒有指定STAGE的JOB是處在「$CI_JOB_STAGE」階段"
      ```
+ `stages`預設是
    ```yml
    stages:
      - .pre
      - build
      - test
      - deploy
      - .post
    ```
> **WARN:** 檔案中允許沒有JOB的`stage`，但不允許JOB使用不使用的`stage`，後者的意思是JOB中所定義的`stage`，不能不出現在上層的`stages`裡面。這種會報文法錯誤，直接推上gitlab來執行管線的話也會失敗。

## 依賴性的`need`關鍵字
目前我只想的到用來約束同stage工作和增加可讀性的用途。
    
