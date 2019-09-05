# 使用SpringDataJpa进行分页
**所需要的maven依赖**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```
## 实际操作
### 通过生成Pageable对象
- Pageable定义了很多方法，但其核心的信息只有两个：一是分页的信息（page、size），二是排序的信息。Spring Data Jpa提供了PageRequest的具体实现，Spring MVC提供了对Spring Data JPA非常好的支持，我们只提供分页以及排序信息即可：
```java
@RequestMapping(value = "/params", method=RequestMethod.GET)
public Page<Blog> getEntryByParams(@RequestParam(value = "page", defaultValue = "0") Integer page,
        @RequestParam(value = "size", defaultValue = "15") Integer size) {
    Sort sort = new Sort(Direction.DESC, "id");
    Pageable pageable = new PageRequest(page, size, sort);
    return blogRepository.findAll(pageable);
}
```
- 在这里，我们通过参数获得分页的信息，并通过Sort以及Direction告诉pageable需要通过id逆序排列。
- 这里可以看到，通过参数来得到一个pageable对象还是比较繁琐的，当查询的方法比较多的时候，会产生大量的重复代码。为了避免这种情况，Spring Data提供了直接生成pageable的方式。
### 通过注解直接获取Pateable对象
```java
@RequestMapping(value = "", method=RequestMethod.GET)
public Page<Blog> getEntryByPageable(@PageableDefault(value = 15, sort = { "id" }, direction = Sort.Direction.DESC) 
    Pageable pageable) {
    return blogRepository.findAll(pageable);
}
```
- 我们可以看到，我们只需要在方法的参数中直接定义一个pageable类型的参数，当Spring发现这个参数时，Spring会自动的根据request的参数来组装该pageable对象，Spring支持的request参数如下：

    - page，第几页，从0开始，默认为第0页
    - size，每一页的大小，默认为20
    - sort，排序相关的信息，以property,property(,ASC|DESC)的方式组织，例如sort=firstname&sort=lastname,desc表示在按firstname正序排列基础上按lastname倒序排列
- 这样，我们就可以通过url的参数来进行多样化、个性化的查询，而不需要为每一种情况来写不同的方法了。

- 通过url来定制pageable很方便，但唯一的缺点是不太美观，因此我们需要为pageable设置一个默认配置，这样很多情况下我们都能够通过一个简洁的url来获取信息了。

- Spring提供了@PageableDefault帮助我们个性化的设置pageable的默认配置。例如@PageableDefault(value = 15, sort = { "id" }, direction = Sort.Direction.DESC)表示默认情况下我们按照id倒序排列，每一页的大小为15。
### 返回结果
**这里直接使用我自己做的项目的结果**
```json
{
    "data": {
            "result": [
                {
                    "id": 455,
                    "showMsg": "2014年927联考：起草一个关于垃圾分类、处理的经验介绍提纲",
                    "type": 1,
                    "essaySimilarQuestionVOList": [
                        {
                            "id": 639,
                            "questionDate": "9-27",
                            "areaName": "四川",
                            "score": 30,
                            "stem": "2014年927四川：垃圾分类介绍提纲"
                        },
                        {
                            "id": 627,
                            "questionDate": "9-27",
                            "areaName": "河南",
                            "score": 20,
                            "stem": "2014年927河南：垃圾分类、处理的介绍提纲"
                        },
                        {
                            "id": 613,
                            "questionDate": "9-27",
                            "areaName": "甘肃",
                            "score": 20,
                            "stem": "2014年927甘肃：垃圾分类、处理的介绍提纲"
                        }
                    ]
                }
            ]
        "next": 1,
        "total": 67,
        "totalPage": 4
    },
    "code": 1000000
}
```

**在使分页查询的时候要空值返回值的类型为了方便前端进行处理可以将总页数和当前页数等相关信息都封装起来一并返回**

```java
   /**
     * @param pageable 利用注解在参数中定义pageable(美观省事)
     *                 重点:还是需要前台提供相应的属性
     *                 size:每页的查询数量
     *                 page:查询的页数
     *                 sort:用来对查询的结果进行排序
     * @return         分页之后的结果
     */
    @GetMapping(value = "list", produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
    public PageUtil<List<EssaySimilarQuestionGroupVO>> list(@PageableDefault(size = 20, sort = {"id"}, direction = Sort.Direction.DESC) Pageable pageable) {
        //利用SpringDate jpa 提供的方法来完成分页:  第三个参数是用来排序的(通过id降序排列返回数据)
        return essayStatisticsService.findAllGroup(pageable);
    }
```
**在使用jpa查询的时候就可以将pageaple对象一并传递过去这样结果就会自动的进行分页我们也可以从其中获得到相关的数据**

```java
    public PageUtil<List<EssaySimilarQuestionGroupVO>> findAllGroup(Pageable pageable) {
        //组装数据
        Page<List<EssaySimilarQuestionGroupInfo>> essaySimilarQuestionGroupInfos = essaySimilarQuestionGroupInfoRepository.findByBizStatusAndStatus(EssayStatisticsConstant.EssayStatisticsStatus.ONLINE.getStatus(), EssayStatisticsConstant.EssayStatisticsBizStatus.NORMAL.getBizSatus(), pageable);
```
**对返回值的结果进行封装,在封装的时候也要进行校验防止有错误的数据传到前台**
```java
    /**
         * getTotalElements     得到元素总数
         * getPageNumber        得到要返回的页数
         * getSize              得到每页总共有多少信息
         */
        long totalElements = essaySimilarQuestionGroupInfos.getTotalElements();
        int pageNumber = pageable.getPageNumber();
        int pageSize = pageable.getPageSize();
        //对分页结果进行封装
        PageUtil resultPageUtil = PageUtil.builder().result(essaySimilarQuestionGroupVOS)
                .total(totalElements)
                .totalPage(0 == totalElements % pageSize ? totalElements / pageSize : totalElements / pageSize + 1)
                .next(totalElements > pageSize * pageNumber ? 1 : 0)
                .build();
        return resultPageUtil;
```