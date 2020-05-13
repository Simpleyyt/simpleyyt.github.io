---
layout: post
categories: 算法
title: 明明谢谢参与的大小跟特等奖的大小一样，可是为什么我们大部分情况还是只能中谢谢参与
tags:
  - 子悠
---

Hello 大家好，我是鸭血粉丝，不知道你有没有过这样的经历，经常在手机上参与各种抽奖，但是明明眼看着就要中特等奖了，但是最后还是中了个谢谢参与，阿粉我就是一个中奖绝缘体，以前总是觉得是自己的运气不好，但是自从知道了背后的暗箱操作过后，阿粉就见怪不怪了。

![img](http://www.justdojava.com/assets/images/2019/java/image_ziyou/weight01.png)

<!--more-->

#### 背景

前段时间阿粉参与了一个活动的项目，这个项目有个功能就是在最后会有一个抽奖的页面，用户只要完成最后一步就能参与抽奖，并且奖品有 IPad 等大奖。但是大奖必定是有限制的，总不能无终止的让用户抽取，原本抽奖都是随机的，但是这种情况下肯定不能完全随机，因为要按照奖品的数量进行伪随机。

让数量多的小奖品更容易被抽中，数量少的大奖品被抽中的概率变小，那么在这种情况下，我们可以很自然的想到，我们需要给奖品增加一个权重来满足这个要求了。

#### 设计

我们假设某次抽奖的奖品列表为”谢谢参与“，”50 元话费“，”200 元京东购物卡“，”Java 极客技术知识星球年卡一张“。然后因为经费有限，我们 50 元话费的名额为 100 个，200 元购物卡的 50 个，知识星球年卡 10 个，当然谢谢参与无限量。按照这种情况，我们就需要给每个奖品设置响应的权重值，假设我们这里这几个奖品对应的权重依次为 99，0.7，0.2，0.1，说明中谢谢参与的权重为 99，中年卡的权重为 0.1，为了方便计算我们这里将权重全部放大十倍，调整为 990，7，2，1。

![img](http://www.justdojava.com/assets/images/2019/java/image_ziyou/weight02.png)

##### 思考

为了实现按照权重随机的效果，我们可以把权重想象成x 坐标轴上的位置，在 x 轴上标出每个奖品的范围，如下图所示：

![](http://www.justdojava.com/assets/images/2019/java/image_ziyou/weight03.png)

从上图我们可以看出只要范围在 [0, 990) 的就说明中奖谢谢参与，同理 [990, 997) 表示中奖 50 元话费，[997, 999) 表示中奖 200 元京东购物卡，[999, 1000) 表示中奖知识星球年卡一张。

按照这种思路，我们只要随机生成一个数，然后看这个数在哪个范围里面，就能知道中的是什么奖品，而且这个中奖的比例也是跟权重正相关的。根据这个方案，我们需要记录这么几个数，一个是奖品 key，一个是权重 weight，一个是范围的最低值以及范围的最大值。

定义一个类用来存放上面提到的四个属性，如下

```java
		@Data
    @NoArgsConstructor
    @AllArgsConstructor
    public static class WeightNode {
        /**
         * 奖品
         **/
        private String key;
        /**
         * 奖品权重
         **/
        private Integer weight;
        /**
         * 最低范围
         **/
        private Integer minValue;
        /**
         * 最高范围
         **/
        private Integer maxValue;

        @Override
        public String toString() {
            return "key:" + this.key + " weight:" + this.weight + " low:" + this.minValue + " height:" + this.maxValue;
        }
    }
```

在定义一个使用的类

```java
public class WeightRandom {

    /**
     * 存储数据节点
     */
    private List<WeightNode> nodes;

    /**
     * 设置数据与权重
     *
     * @param keys    数据
     * @param weights 权重
     */
    public void addNode(String[] keys, Integer[] weights) {
        if (keys == null || weights == null || keys.length != weights.length) {
            return;
        }
        nodes = new ArrayList<>();
        for (int i = 0; i < keys.length; i++) {
            Integer minValue = 0;
            Integer maxValue = 0;
            if (i == 0) {
                minValue = i;
                maxValue = weights[i];
            } else {
                if (i + 1 <= keys.length) {
                    Integer sum = preSum(weights, i);
                    minValue = sum;
                    maxValue = sum + weights[i];
                }
            }
            nodes.add(new WeightNode(keys[i], weights[i], minValue, maxValue));
        }
    }

    /**
     * 累加权重
     *
     * @param array 数组
     * @param index 索引
     * @return 和
     */
    private Integer preSum(Integer[] array, Integer index) {
        Integer sum = 0;
        for (int i = 0; i < array.length; i++) {
            if (i < index) {
                sum += array[i];
            }
        }
        return sum;
    }

    /**
     * 根据生成的随机数遍历所有元素，查找到随机数的范围
     *
     * @param randomValue 生成的随机数
     * @return 范围节点
     */
    public WeightNode getElementByRandomValue(Integer randomValue) {
        //左闭右开
        for (WeightNode e : nodes) {
            if (randomValue >= e.getMinValue() && randomValue < e.getMaxValue()) {
                return e;
            }
        }
        return null;
    }

    /**
     * 在权重是顺序的情况下，可以采用二分法加速查询
     *
     * @param randomValue 生成的随机数
     * @return 范围节点
     */
    public WeightNode getElementByRandomValueBinary(Integer randomValue) {
        if (randomValue < 0 || randomValue > getMaxRandomValue() - 1) {
            return null;
        }
        int start = 0, end = nodes.size() - 1;
        int middle = nodes.size() >> 1;
        while (true) {
            if (randomValue <= nodes.get(middle).getMinValue()) {
                end = middle - 1;
            } else if (randomValue > nodes.get(middle).getMaxValue()) {
                start = middle + 1;
            } else {
                return nodes.get(middle);
            }
            middle = (start + end) >> 1;
        }
    }

    /**
     * 权重总和，用于设置生成随机数的最大上限
     *
     * @return 权重总和
     */
    public Integer getMaxRandomValue() {
        if (CollectionUtils.isEmpty(nodes)) {
            return null;
        }
        return nodes.get(nodes.size() - 1).getMaxValue();
    }

    public static void main(String[] args) {
        WeightRandom wr = new WeightRandom();
//        wr.initWeight(new String[]{"谢谢参与", "50 元话费", "200 元京东购物卡", "星球年卡一张"}, new Integer[]{10, 10, 10, 70});
        wr.addNode(new String[]{"谢谢参与", "50 元话费", "200 元京东购物卡", "星球年卡一张"}, new Integer[]{990, 7, 2, 1});
        Random random = new Random();
        HashMap<String, Integer> keyCount = new HashMap<>(16);
        keyCount.put("谢谢参与", 0);
        keyCount.put("50 元话费", 0);
        keyCount.put("200 元京东购物卡", 0);
        keyCount.put("星球年卡一张", 0);
        for (int i = 0; i < 100000; i++) {
            Integer randomValue = random.nextInt(wr.getMaxRandomValue());
            String key = wr.getElementByRandomValue(randomValue).getKey();
            keyCount.put(key, keyCount.get(key) + 1);
        }
        keyCount.forEach((key, value) -> {
            System.out.println(key + ": " + value);
        });
    }
}

```

输出的结果：

```java
谢谢参与: 99016
Java极客技术知识星球年卡一张: 94
50 元话费: 696
200 元京东购物卡: 194

Process finished with exit code 0

```

通过输出的结果我们看到，基本是按照我们设置的权重进行随机的。通过上面的示例阿粉带大家设计了一个带权重的随机实现方案，除了在抽奖的场景会使用的，在其他类似的场景也可以使用。

#### 参考

https://yq.aliyun.com/articles/269456

### 写在最后

---

最后邀请你加入我们的知识星球，这里有 1700+ 优秀的人与你一起进步，如果你是小白那你是稳赚了，很多业内经验和干活分享给你；如果你是大佬，那可以进来我们一起交流分享你的经验，说不定日后我们还可以有合作，给你的人生多一个可能。

![image-20200205235453492](http://www.justdojava.com/assets/images/2019/java/image_ziyou/子悠-知识星球.png)

