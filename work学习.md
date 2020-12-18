

##### 阿里云下载文件改名

```javascript
* getExportManagePage({payload}, {call, put}) {
    const data = yield call(exportManagePage, payload)
    const ossStsResult = yield call(getOssSts, {});// 获取阿里云授权

    if (!ossStsResult) return;

    const client = new OSS({
        region: 'oss-cn-shenzhen',
        accessKeyId: ossStsResult.data.accessKeyId,
        accessKeySecret: ossStsResult.data.accessKeySecret,
        stsToken: ossStsResult.data.stsToken,
        bucket: ossStsResult.data.bucket,
        secure: true
    });

    yield put ({
        type: 'setState',
        payload: {
            exportManageDataSource : data && data.list ? data.list.map((item) => {
                item.url = client.signatureUrl(item.path || '', {response: {'content-disposition': `attachment; filename=${item.name}`}});
                item.key = item.id
                return item
            }) : []
        }
    })
}
```