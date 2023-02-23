# Laravel プロジェクト用の pre-commit を用意する

## 参考記事

- [PHPer 向けの pre-commit で静的解析ライフ](https://qiita.com/nsym__m/items/35940f66de4cb45056ab)

## 完成フォーマット

```sh
#!/bin/sh

if git rev-parse --verify HEAD >/dev/null 2>&1
then
	against=HEAD
else
	# Initial commit: diff against an empty tree object
	against=`git hash-object -t tree /dev/null`
fi

phpfiles=`git diff --name-only --diff-filter=d $against | grep \.php`
if [ "$phpfiles" != "" ]; then

    printf '####\033[32m check phpstan \033[m####\n'
    printf '####\033[32m check php-cs-fixer \033[m####\n'
    printf '####\033[32m check php -l \033[m####\n'
    echo

    # ディレクトリ定義
    ROOT_DIR=`git rev-parse --show-toplevel`

    # エラー定義
    errorStan=false
    errorCs=false
    errorL=false
    for file in `git diff --cached --name-only --diff-filter=ACM | grep \.php`
    do
        # phpstan実行 全ファイルにチェックすると既存ファイルにエラーがある時コミットできないので、コミット対象のファイルのみチェックする
        phpstanCheck=`./vendor/bin/phpstan analyze --memory-limit=2G $ROOT_DIR/$file`
        # エラーがなければ'No errors'という文字列があるので、それがない場合にエラーと判断
        echo $phpstanCheck | grep 'No errors' > /dev/null 2>&1
        # $?には直前の実行結果が入る
        if [ $? != 0 ]; then
            printf "####\033[31m　phpstan failed!!!! \033[m####$phpstanCheck\n"
            errorStan=true
        else
            # php -lによるシンタックスチェック実行 app/配下以外だとphpstanが実行されない気がする？為入れてる
            syntaxCheck=`php -l $file`
            # エラーがなければ'No syntax errors'という文字列があるので、それがない場合にエラーと判断
            echo $syntaxCheck | grep 'No syntax errors' > /dev/null 2>&1
            if [ $? != 0 ]; then
                # シンタックスエラーのあった場合はエラー内容を出力
                printf "####\033[31m php -l failed!!!! \033[m####$syntaxCheck\n"
                errorL=true
            fi
        fi

        # php-cs-fixer実行 --dry-run で修正はせずに引っ掛かったらコミットさせずに修正を促す
        # --path-modeをintersectionにすることで、Finderが無視されないようにする
        .tools/php-cs-fixer/vendor/bin/php-cs-fixer fix --path-mode=intersection --dry-run $ROOT_DIR/$file > /dev/null 2>&1
        if [ $? != 0 ]; then
            printf "####\033[31m php-cs-fixer failed!!!! \033[m####\n $ROOT_DIR/$file\n"
            errorCs=true
        fi
    done

    # "phpstan", "php-cs-fixer", "php -l"のどれかに引っ掛かっていたらコミットを中断
    if "${errorStan}"; then
        printf "####\033[31m　Commit fail\033[m please fix \033[31mphpstan errors\033[m ####\n"
    fi
    if "${errorL}"; then
        printf "####\033[31m　Commit fail\033[m please \033[31msyntax check\033[m ####\n"
    fi
    if "${errorCs}"; then
        printf "####\033[31m　Commit fail\033[m please run \033[31m\"tools/php-cs-fixer/vendor/bin/php-cs-fixer fix\"\033[m command ####\n"
    fi
    if [ $errorStan || $errorL || $errorCs ]; then
        printf "コミットに失敗しました。チェックの見直しを行ってください。"
        exit 1
    fi

    printf '####\033[32m phpstan, php -l, php-cs-fixer all completed!! \033[m####\n'

fi
```
