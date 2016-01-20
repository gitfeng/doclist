---
title: php_遍历文件内容
date: 2016-01-17 15:41:00
categories: 技术
tags: [php]
description: php 遍历文件内容，然后可执行其他任务，脚本备份

---

```php
<?php
/** 
 * first finish doLogic function,
 * then run as 'php script.php filepath'
 */

BD_Init::init();
Class FileDeal {
    function __construct($file) {
        $this->filePath = $file;
    }
    public function run() {
        $i = 0;
        $data = array();
        $file = fopen($this->filePath, "r");
        while(($line = fgets($file)) !== false){
            $arrQa = explode("\t", $line);
            $data[] = $arrQa;
            $i++;
            if ($i %20 == 0) {
                $this->doLogic($data);
                $data = array();
                usleep(200000);
            }
        }
        fclose($file);
    }
    public function doLogic($data) {
        //dologic
    }
}
$file = trim($argv[1]);
$cls = new FileDeal($file);
$cls->run();
```
