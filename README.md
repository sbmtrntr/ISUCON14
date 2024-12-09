# ISUCON14
## チーム名
**MML**
## メンバー
[＠sbmtrntr](https://github.com/sbmtrntr) [@Sonookoo](https://github.com/Sonokoo) [@Mondsichel](https://github.com/Mondsichel)
## 成績
- 最終スコア12549点
- 全体134位
- 学生17位

<details>
  <summary>来年に向けての備忘録</summary>
  
  - 最初にやること
    - .gitignoreとMakefileの取得(実行場所：/home/isucon)
    
        ```bash
        curl https://raw.githubusercontent.com/oribe1115/traP-isucon-newbie-handson2022/main/Makefile -o Makefile
        curl https://raw.githubusercontent.com/oribe1115/traP-isucon-newbie-handson2022/main/.gitignore -o .gitignore
        # Makefileとgitignoreを問題と対応させる
        make setup #ssh-keygenを行っているのでyesを押して進める
        ```
        
    - git管理
      
          ```bash
          cat .ssh/id_ed25519.pub
          #作成したリポジトリのSettings > Deploy Keysでサーバの公開鍵を登録する
          git init
          git remote add origin {リポジトリのURL}
          git add .
          git commit -m "init"
          git branch -m main
          git push origin main
          ```
      
- 計測ツールの設定
    - alp
        - config.ymlの作成(実行場所：/home/isucon)
          
        `mkdir -p tool-config/alp && sudo nano tool-config/alp/config.yml`
      
        - config.ymlの中身
        ```yml
        ---
        sort: sum                      # max|min|avg|sum|count|uri|method|max-body|min-body|avg-body|sum-body|p1|p50|p99|stddev
        reverse: true                   # boolean
        query_string: true              # boolean
        output: count,5xx,method,uri,min,max,sum,avg,p99                    # string(comma separated)
        
        matching_groups:            # array
          - /api/admin/tenants/add$ # みたいな正規表現でまとめる
        
        ```
        
        - ログ設定の追加場所：/etc/nginx/nginx.conf
        
        ```
           log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                              '$status $body_bytes_sent "$http_referer" '
                              '"$http_user_agent" "$http_x_forwarded_for"';
            
           log_format ltsv "time:$time_local"
                           "\thost:$remote_addr"
                           "\tforwardedfor:$http_x_forwarded_for"
                           "\treq:$request"
                           "\tstatus:$status"
                           "\tmethod:$request_method"
                           "\turi:$request_uri"
                           "\tsize:$body_bytes_sent"
                           "\treferer:$http_referer"
                           "\tua:$http_user_agent"
                           "\treqtime:$request_time"
                           "\tcache:$upstream_http_x_cache"
                           "\truntime:$upstream_http_x_runtime"
                           "\tapptime:$upstream_response_time"
                           "\tvhost:$host";
        
            access_log  /var/log/nginx/access.log  ltsv;
        ```
      - 設定を変更したらnginxを再起動するのを忘れない `sudo systemctl restart nginx`
      - 使い方 `make alp`
        
    - pt-query-digest
        - 設定の追加場所
          - mysqlの場合 `/etc/mysql/mysql.conf.d/mysqld.cnf`
          - mariadbの場合 `/etc/mysql/mariadb.conf.d/50-server.cnf`
        - 設定の追加内容
        ```
        [mysqld]
        slow_query_log=1
        slow_query_log_file=/var/log/mysql/mysql-slow.log
        long_query_time=0
        ```
        - 設定を変更したらmysqlを再起動するのを忘れない`sudo systemctl restart mysql`
        - 使い方 `sudo pt-query-digest /var/log/mysql/mysql-slow.log | tee digest_$(date +%H%M).txt`
        
    - pprof
        - 設定方法 `webapp/go/main.go`のimportとmainに追加
        ```bash
        import (
        	_ "net/http/pprof"
            ...
        )
        
        ...
        
        func main() {
            go func() {
        		log.Fatal(http.ListenAndServe(":6060", nil))
        	}()
            ...
        }
        ```
        - 使い方
        ```bash
        make pprof-record # ベンチマーク中に実行
        make pprof-check #ベンチマーク後、ブラウザで結果を確認
        ```
        
    - golang
        - 変更したらbuildするのを忘れない`make build`

- ベンチマーク

  - ベンチマーク前
      
      ```bash
      make restart
      sudo rm /var/log/mysql/mysql-slow.log
      sudo mysqladmin flush-logs
      ```
      
  - ベンチマーク中
      
      ```bash
      make pprof-record
      ```
      
  - ベンチマーク後
      
      ```bash
      sudo pt-query-digest /var/log/mysql/mysql-slow.log | tee digest_$(date +%Y%m%d%H%M).txt
      make alp
      make pprof-check
      ```
    
- mysql
    
    **ログイン**
    
    ```bash
    sudo mysql -u root
    ```
    
    **データベース一覧確認**
    
    ```sql
    SHOW DATABASES;
    ```
    
    **データベースの構造確認**
    
    ```sql
    SHOW CREATE TABLE <テーブル名>\G;
    ```
    
    - ERROR 1049 (42000): Unknown database 'comments'が出る時
    
    ```sql
    use isuconp;
    show tables; # 対象の表があるか確認
    ```
    
    **クエリの実行動作を確認**
    
    ```sql
    EXPLAIN SELECT * FROM comments WHERE post_id = 9995 ORDER BY created_at DESC LIMIT 3\G
    ```
      
    **post_idカラムにインデックスを付与**
    ```sql
    ALTER TABLE comments ADD INDEX post_id_idx(post_id); # CREATE INDEXとかもあるらしい
    ```
        
</details>
