
``` go
//断点续传流程
func resume_put(file,scope){
	
	// 以4Mb大小为单位，将文件切割成块（blocks）
	// 是后一个块的大小为 file.size - (n-1)*1 << 22
	blocks[] = file.mkblk(1<<22)

	//blkRet为上传块时七牛返回的数据结构,格式如下:
	//{
	//	"ctx":	"MWZvQbq10x...",
	//	"crc32":1957222263,
	//	"offset":2097152,
	//	"host":"http://upbeta.qiniu.com"
	//}
	//blkRet的个数与切割块的数目相同
	blkRet[blocks.len]

	//当前上传块在数组blocks中的索引号
	blkIdx = 0

	foreach(blk in blocks){

		//上传块，此逻辑可并发执行
		//@block, 需要上传的快
		fun(block){

			//以256Kb为单位，将block切割为chunk数组
			chunks[] = block.chunk(256)

			//通知Qiniu，开始上传block，同时将block的第一个chunk上传至七牛
			//@blockSize, 上传块的大小.(除最后一个块，其余为4Mb)
			//@furstChunk, block切割成的chunk数组的第一个元素
			fun qiniu_mkblk(blockSize,firstChunk){
				url = "http://up.qiniu.com/mkblk/"+blockSize
				blkRet[blkIdx] = httpClient.send(url,firstChunk)
			}(chunks[0]);

			//继续上传余下的chunk,注意i=1,表明跳过了首个chunk,
			//因为此chunk已经在mkblk时上传了
			
			for(i=1;i<chunks.len;i++){

				//上传chunks
				//@host, 接收上传的地址，可以从上一次返回结果中获取
				//@ctx, 用于上传控制，从上一次上传返回结果中获取
				//@offset, chunk在block中的偏移量，以byte为单位，可以从上一次返回结果中获取
				//@chunk, 需要上传的chunk

				fun qiniu_bput(host,ctx,offset,chunk){

					//请求地址
					url = host + "/bput/" + ctx + "/" +offset 

					//blkRet的内容被替换
					blkRet[blkIdx] = httpClient.send(url,chunk);

				}(blkRet[blkIdx].host,blkRet[blkIdx].ctx,blkRet[blkIdx].offset,chunks[i])

			}

		}(blk)

		blkIdx++
	}

	//所有block上传完成，调用mkfile请求在服务端生成完整文件
	//@file 上传的文件
	//@scope, bucket + ":" + key
	//@return ,上传返回结果，默认为{hash:<hash>,key:<key>}
	return fun mkfile(file,scope){
		
		//生成mkfile请求地址
		url = "http://up.qiniu.com/rs-mkfile/" +
				base64Safe(scope) +                // 必须
				"/fsize/" + file.size +            // 必须
				"/mimeTpye/" + base64Safe(file.type) // 可选
		
		foreach(ret in blkRet){
			body += ret.ctx + ","
		}

		body.TrimEnd(",")
		return httpClient.send(url, body)
	}(file,scope)

}
```
