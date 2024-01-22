需求：在开启S3 Version的情况下，使用命令行递归遍历全部删除子目录和子文件，从而完成全部恢复。

操作方法：

1、需要恢复的S3根目录是：
https://cloudrenderingsolution-20211225.s3.cn-northwest-1.amazonaws.com.cn/install/blender/3.0/
请注意， 下面的名字需要在代码里面进行替换为你的实际名字

- S3桶是：cloudrenderingsolution-20211225，

- 需要恢复的（根级）目录是：BUCKETNAME/install/blender/3.0/

  

2、查看需要恢复的子目录和文件的清单：
  请注意：'2024-01-22' 是文件删除的日期；

``` bash
aws s3api list-object-versions --bucket cloudrenderingsolution-20211225  --prefix install/blender/3.0/ --query "DeleteMarkers[?IsLatest && starts_with(LastModified,'2024-01-22')].{Key:Key,VersionId:VersionId}"
```


3、确认删除文件无误后，请执行下面的代码，完成文件的恢复

``` bash
recoverfiles=$(aws s3api list-object-versions --bucket cloudrenderingsolution-20211225  --prefix install/blender/3.0/ --query "DeleteMarkers[?IsLatest && starts_with(LastModified,'2024-01-22')].{Key:Key,VersionId:VersionId}")
for row in  $(echo "${recoverfiles}" | jq -c '.[]'); do
    key=$(echo "${row}" | jq -r '.Key'  )
    versionId=$(echo "${row}" | jq -r '.VersionId'  )
    echo aws s3api delete-object --bucket cloudrenderingsolution-20211225 --key $key --version-id $versionId 
    aws s3api delete-object --bucket cloudrenderingsolution-20211225 --key $key --version-id $versionId > /dev/null 2>&1
done
```



4、恢复的原理：
使用 AWS CLI 删除了《删除标记》，所以被删除的文件就自然恢复了！
因为取消删除可以通过两种方式完成：
A、删除将反转删除的删除标记；
B、将对象的先前版本复制到自身，这将使最新版本比删除标记更新，因此它将重新出现。（希望你能明白）

![image-20240122152011164](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20240122152011164.png)



参考ref：
How can I retrieve an Amazon S3 object that was deleted in a versioning-enabled bucket?
https://repost.aws/knowledge-center/s3-undelete-configuration

Undelete folders from AWS S3
https://stackoverflow.com/questions/45207055/undelete-folders-from-aws-s3
