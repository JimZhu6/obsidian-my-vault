# php常见抽奖算法



## 奖品设置

```php
$award = [{id: "1", award: "幸运值+5", probability: "48", quantity: "0"},
        {id: "2", award: "幸运值+1", probability: "50", quantity: "0"},
        {id: "3", award: "iPad pro", probability: "0", quantity: "0"},
        {id: "4", award: "联想笔记本电脑", probability: "0", quantity: "0"}]
```





## 根据概率随机抽奖项

```php
/**
 * 抽奖逻辑
 */
private function randPrize($award)
{
    $result = array();
    foreach ($award as $key => $val) {
        // 1.每个抽奖项的获取概率
        $arr[$key] = $val['probability'];
    }
    // 2.计算总概率
    $proSum = array_sum($arr);
    // 3.重排概率
    asort($arr);
    // 4.概率数组循环
    foreach ($arr as $k => $v) {
        $randNum = mt_rand(1, $proSum);
        if ($randNum <= $v) {
            $result = $award[$k];
            break;
        } else {
            $proSum -= $v;
        }
    }
    return $result['id'];
}
```



