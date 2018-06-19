---
layout: post
title:  "比特币 RPC 命令剖析 \"getblocktemplate\""
date:   2018-06-19 09:10:18 +0800
author: mistydew
categories: Blockchain
---
![bitcoin](/images/20180504/bitcoin.svg)

## 读在前面
比特币相关的解读目前均采用 [bitcoin v0.12.1](https://github.com/bitcoin/bitcoin/tree/v0.12.1)，此版本为官方内置挖矿算法的最后一版。<br>
目前比特币的最新版本为 bitcoin v0.16.0，离区块链 1.0 落地还有些距离。

## 提示说明

{% highlight shell %}
getblocktemplate ( "jsonrequestobject" ) # 获取一个区块模板
{% endhighlight %}

如果请求参数包含 `mode` 关键字，用于在默认的 `template` 请求或 `proposal` 间选择。
返回构建一个区块所需的数据。
完整规范详见 [https://en.bitcoin.it/wiki/BIP_0022](https://en.bitcoin.it/wiki/BIP_0022)。

参数：<br>
1. `jsonrequestobject` （字符串，可选）以下规范中的 json 对象。<br>
{% highlight shell %}
     {
       "mode":"template"    (string, optional) This must be set to "template" or omitted
       "capabilities":[       (array, optional) A list of strings
           "support"           (string) client side supported feature, 'longpoll', 'coinbasetxn', 'coinbasevalue', 'proposal', 'serverlist', 'workid'
           ,...
         ]
     }
{% endhighlight %}

结果：<br>
{% highlight shell %}
{
  "version" : n,                    (numeric) The block version
  "previousblockhash" : "xxxx",    (string) The hash of current highest block
  "transactions" : [                (array) contents of non-coinbase transactions that should be included in the next block
      {
         "data" : "xxxx",          (string) transaction data encoded in hexadecimal (byte-for-byte)
         "hash" : "xxxx",          (string) hash/id encoded in little-endian hexadecimal
         "depends" : [              (array) array of numbers 
             n                        (numeric) transactions before this one (by 1-based index in 'transactions' list) that must be present in the final block if this one is
             ,...
         ],
         "fee": n,                   (numeric) difference in value between transaction inputs and outputs (in Satoshis); for coinbase transactions, this is a negative Number of the total collected block fees (ie, not including the block subsidy); if key is not present, fee is unknown and clients MUST NOT assume there isn't one
         "sigops" : n,               (numeric) total number of SigOps, as counted for purposes of block limits; if key is not present, sigop count is unknown and clients MUST NOT assume there aren't any
         "required" : true|false     (boolean) if provided and true, this transaction must be in the final block
      }
      ,...
  ],
  "coinbaseaux" : {                  (json object) data that should be included in the coinbase's scriptSig content
      "flags" : "flags"            (string) 
  },
  "coinbasevalue" : n,               (numeric) maximum allowable input to coinbase transaction, including the generation award and transaction fees (in Satoshis)
  "coinbasetxn" : { ... },           (json object) information for coinbase transaction
  "target" : "xxxx",               (string) The hash target
  "mintime" : xxx,                   (numeric) The minimum timestamp appropriate for next block time in seconds since epoch (Jan 1 1970 GMT)
  "mutable" : [                      (array of string) list of ways the block template may be changed 
     "value"                         (string) A way the block template may be changed, e.g. 'time', 'transactions', 'prevblock'
     ,...
  ],
  "noncerange" : "00000000ffffffff",   (string) A range of valid nonces
  "sigoplimit" : n,                 (numeric) limit of sigops in blocks
  "sizelimit" : n,                  (numeric) limit of block size
  "curtime" : ttt,                  (numeric) current timestamp in seconds since epoch (Jan 1 1970 GMT)
  "bits" : "xxx",                 (string) compressed target of next block
  "height" : n                      (numeric) The height of the next block
}
{% endhighlight %}

## 用法示例

该命令需要请求的节点至少建立一条连接。

{% highlight shell %}
$ bitcoin-cli getblocktemplate
{% endhighlight %}

## 源码剖析
`getblocktemplate` 对应的函数在“rpcserver.h”文件中被引用。

{% highlight C++ %}
extern UniValue getblocktemplate(const UniValue& params, bool fHelp); // 获取区块模板
{% endhighlight %}

实现在“rpcmining.cpp”文件中。

{% highlight C++ %}
UniValue getblocktemplate(const UniValue& params, bool fHelp)
{
    if (fHelp || params.size() > 1) // 参数最多为 1 个
        throw runtime_error( // 命令帮助反馈
            "getblocktemplate ( \"jsonrequestobject\" )\n"
            "\nIf the request parameters include a 'mode' key, that is used to explicitly select between the default 'template' request or a 'proposal'.\n"
            "It returns data needed to construct a block to work on.\n"
            "See https://en.bitcoin.it/wiki/BIP_0022 for full specification.\n"

            "\nArguments:\n"
            "1. \"jsonrequestobject\"       (string, optional) A json object in the following spec\n"
            "     {\n"
            "       \"mode\":\"template\"    (string, optional) This must be set to \"template\" or omitted\n"
            "       \"capabilities\":[       (array, optional) A list of strings\n"
            "           \"support\"           (string) client side supported feature, 'longpoll', 'coinbasetxn', 'coinbasevalue', 'proposal', 'serverlist', 'workid'\n"
            "           ,...\n"
            "         ]\n"
            "     }\n"
            "\n"

            "\nResult:\n"
            "{\n"
            "  \"version\" : n,                    (numeric) The block version\n"
            "  \"previousblockhash\" : \"xxxx\",    (string) The hash of current highest block\n"
            "  \"transactions\" : [                (array) contents of non-coinbase transactions that should be included in the next block\n"
            "      {\n"
            "         \"data\" : \"xxxx\",          (string) transaction data encoded in hexadecimal (byte-for-byte)\n"
            "         \"hash\" : \"xxxx\",          (string) hash/id encoded in little-endian hexadecimal\n"
            "         \"depends\" : [              (array) array of numbers \n"
            "             n                        (numeric) transactions before this one (by 1-based index in 'transactions' list) that must be present in the final block if this one is\n"
            "             ,...\n"
            "         ],\n"
            "         \"fee\": n,                   (numeric) difference in value between transaction inputs and outputs (in Satoshis); for coinbase transactions, this is a negative Number of the total collected block fees (ie, not including the block subsidy); if key is not present, fee is unknown and clients MUST NOT assume there isn't one\n"
            "         \"sigops\" : n,               (numeric) total number of SigOps, as counted for purposes of block limits; if key is not present, sigop count is unknown and clients MUST NOT assume there aren't any\n"
            "         \"required\" : true|false     (boolean) if provided and true, this transaction must be in the final block\n"
            "      }\n"
            "      ,...\n"
            "  ],\n"
            "  \"coinbaseaux\" : {                  (json object) data that should be included in the coinbase's scriptSig content\n"
            "      \"flags\" : \"flags\"            (string) \n"
            "  },\n"
            "  \"coinbasevalue\" : n,               (numeric) maximum allowable input to coinbase transaction, including the generation award and transaction fees (in Satoshis)\n"
            "  \"coinbasetxn\" : { ... },           (json object) information for coinbase transaction\n"
            "  \"target\" : \"xxxx\",               (string) The hash target\n"
            "  \"mintime\" : xxx,                   (numeric) The minimum timestamp appropriate for next block time in seconds since epoch (Jan 1 1970 GMT)\n"
            "  \"mutable\" : [                      (array of string) list of ways the block template may be changed \n"
            "     \"value\"                         (string) A way the block template may be changed, e.g. 'time', 'transactions', 'prevblock'\n"
            "     ,...\n"
            "  ],\n"
            "  \"noncerange\" : \"00000000ffffffff\",   (string) A range of valid nonces\n"
            "  \"sigoplimit\" : n,                 (numeric) limit of sigops in blocks\n"
            "  \"sizelimit\" : n,                  (numeric) limit of block size\n"
            "  \"curtime\" : ttt,                  (numeric) current timestamp in seconds since epoch (Jan 1 1970 GMT)\n"
            "  \"bits\" : \"xxx\",                 (string) compressed target of next block\n"
            "  \"height\" : n                      (numeric) The height of the next block\n"
            "}\n"

            "\nExamples:\n"
            + HelpExampleCli("getblocktemplate", "")
            + HelpExampleRpc("getblocktemplate", "")
         );

    LOCK(cs_main); // 上锁

    std::string strMode = "template"; // 模式，默认为 "template"
    UniValue lpval = NullUniValue;
    if (params.size() > 0) // 指定了参数
    {
        const UniValue& oparam = params[0].get_obj(); // 获取参数对象
        const UniValue& modeval = find_value(oparam, "mode"); // 获取 "mode" 关键字对应的值
        if (modeval.isStr()) // 字符串类型
            strMode = modeval.get_str(); // 获取指定模式
        else if (modeval.isNull()) // 空
        {
            /* Do nothing */
        }
        else // 其它类型
            throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid mode");
        lpval = find_value(oparam, "longpollid");

        if (strMode == "proposal") // "proposal" 模式
        {
            const UniValue& dataval = find_value(oparam, "data"); // 获取数据
            if (!dataval.isStr())
                throw JSONRPCError(RPC_TYPE_ERROR, "Missing data String key for proposal");

            CBlock block;
            if (!DecodeHexBlk(block, dataval.get_str())) // 解码 16 进制的区块
                throw JSONRPCError(RPC_DESERIALIZATION_ERROR, "Block decode failed");

            uint256 hash = block.GetHash(); // 获取区块哈希
            BlockMap::iterator mi = mapBlockIndex.find(hash); // 在区块索引列表中查找指定区块
            if (mi != mapBlockIndex.end()) { // 若找到
                CBlockIndex *pindex = mi->second; // 获取指定区块索引指针
                if (pindex->IsValid(BLOCK_VALID_SCRIPTS)) // 验证区块
                    return "duplicate";
                if (pindex->nStatus & BLOCK_FAILED_MASK) // 区块状态
                    return "duplicate-invalid";
                return "duplicate-inconclusive";
            } // 若未找到

            CBlockIndex* const pindexPrev = chainActive.Tip(); // 获取激活链尖
            // TestBlockValidity only supports blocks built on the current Tip
            if (block.hashPrevBlock != pindexPrev->GetBlockHash()) // 指定区块的前一个区块哈希是否为当前链尖区块
                return "inconclusive-not-best-prevblk";
            CValidationState state;
            TestBlockValidity(state, Params(), block, pindexPrev, false, true); // 测试区块有效性
            return BIP22ValidationResult(state); // 返回验证结果
        }
    }

    if (strMode != "template") // "template" 模式
        throw JSONRPCError(RPC_INVALID_PARAMETER, "Invalid mode");

    if (vNodes.empty()) // 已建立连接的节点列表非空
        throw JSONRPCError(RPC_CLIENT_NOT_CONNECTED, "Bitcoin is not connected!");

    if (IsInitialBlockDownload()) // 检查是否初始化块下载完成
        throw JSONRPCError(RPC_CLIENT_IN_INITIAL_DOWNLOAD, "Bitcoin is downloading blocks...");

    static unsigned int nTransactionsUpdatedLast;

    if (!lpval.isNull())
    { // 等待响应，直到最佳块改变，或 1 分钟过去有更多的交易
        // Wait to respond until either the best block changes, OR a minute has passed and there are more transactions
        uint256 hashWatchedChain;
        boost::system_time checktxtime;
        unsigned int nTransactionsUpdatedLastLP;

        if (lpval.isStr())
        {
            // Format: <hashBestChain><nTransactionsUpdatedLast>
            std::string lpstr = lpval.get_str();

            hashWatchedChain.SetHex(lpstr.substr(0, 64));
            nTransactionsUpdatedLastLP = atoi64(lpstr.substr(64));
        }
        else
        { // 注：规范没有对非字符串的 longpollip 指定行为，但这使测试更加容易
            // NOTE: Spec does not specify behaviour for non-string longpollid, but this makes testing easier
            hashWatchedChain = chainActive.Tip()->GetBlockHash(); // 获取链尖区块哈希
            nTransactionsUpdatedLastLP = nTransactionsUpdatedLast; // 最新的交易更新数量
        }

        // Release the wallet and main lock while waiting
        LEAVE_CRITICAL_SECTION(cs_main); // 在等待时释放钱包和主锁
        {
            checktxtime = boost::get_system_time() + boost::posix_time::minutes(1); // 检查交易时间为 1 分钟后

            boost::unique_lock<boost::mutex> lock(csBestBlock); // 最佳区块上锁
            while (chainActive.Tip()->GetBlockHash() == hashWatchedChain && IsRPCRunning())
            { // 最佳区块未改变 且 RPC 服务开启
                if (!cvBlockChange.timed_wait(lock, checktxtime)) // 超时：检查交易用于更新
                {
                    // Timeout: Check transactions for update
                    if (mempool.GetTransactionsUpdated() != nTransactionsUpdatedLastLP)
                        break;
                    checktxtime += boost::posix_time::seconds(10); // 检查时间加 10 秒
                }
            }
        }
        ENTER_CRITICAL_SECTION(cs_main);

        if (!IsRPCRunning()) // 检查 RPC 服务是否开启
            throw JSONRPCError(RPC_CLIENT_NOT_CONNECTED, "Shutting down");
        // TODO: Maybe recheck connections/IBD and (if something wrong) send an expires-immediately template to stop miners?
    }

    // Update block // 更新区块
    static CBlockIndex* pindexPrev;
    static int64_t nStart;
    static CBlockTemplate* pblocktemplate;
    if (pindexPrev != chainActive.Tip() || // 最佳区块非空 或
        (mempool.GetTransactionsUpdated() != nTransactionsUpdatedLast && GetTime() - nStart > 5)) // 交易内存池交易更新数量不等于最近交易更新数 且 当前时间过去 5 秒
    { // 清空 pindexPrev 以便将来调用创建一个新块，尽管这里可能会失败
        // Clear pindexPrev so future calls make a new block, despite any failures from here on
        pindexPrev = NULL; // 置空

        // Store the pindexBest used before CreateNewBlock, to avoid races
        nTransactionsUpdatedLast = mempool.GetTransactionsUpdated(); // 获取当前交易更新数
        CBlockIndex* pindexPrevNew = chainActive.Tip(); // 获取链尖索引
        nStart = GetTime();

        // Create new block
        if(pblocktemplate) // 若区块模板已存在
        {
            delete pblocktemplate; // 先删除
            pblocktemplate = NULL; // 在置空
        }
        CScript scriptDummy = CScript() << OP_TRUE; // 脚本
        pblocktemplate = CreateNewBlock(Params(), scriptDummy); // 创建一个新块
        if (!pblocktemplate)
            throw JSONRPCError(RPC_OUT_OF_MEMORY, "Out of memory");

        // Need to update only after we know CreateNewBlock succeeded
        pindexPrev = pindexPrevNew; // 在我们直到创建新块成功后需要更新前一个区块的哈希
    }
    CBlock* pblock = &pblocktemplate->block; // pointer for convenience

    // Update nTime
    UpdateTime(pblock, Params().GetConsensus(), pindexPrev); // 更新时间
    pblock->nNonce = 0; // 初始化随机数

    UniValue aCaps(UniValue::VARR); aCaps.push_back("proposal");

    UniValue transactions(UniValue::VARR);
    map<uint256, int64_t> setTxIndex; // 交易索引映射列表
    int i = 0;
    BOOST_FOREACH (const CTransaction& tx, pblock->vtx) { // 遍历区块交易索引列表
        uint256 txHash = tx.GetHash(); // 获取交易哈希
        setTxIndex[txHash] = i++; // 加入交易索引映射列表

        if (tx.IsCoinBase()) // 若为创币交易
            continue; // 跳过

        UniValue entry(UniValue::VOBJ);

        entry.push_back(Pair("data", EncodeHexTx(tx))); // 编码 16 进制的交易

        entry.push_back(Pair("hash", txHash.GetHex())); // 获取 16 进制的交易索引

        UniValue deps(UniValue::VARR);
        BOOST_FOREACH (const CTxIn &in, tx.vin) // 遍历交易输入列表
        {
            if (setTxIndex.count(in.prevout.hash)) // 若前一笔交易输出在交易索引映射列表中
                deps.push_back(setTxIndex[in.prevout.hash]); // 加入依赖 json 数组
        }
        entry.push_back(Pair("depends", deps)); // 依赖交易

        int index_in_template = i - 1; // 当前交易的索引序号
        entry.push_back(Pair("fee", pblocktemplate->vTxFees[index_in_template])); // 交易费
        entry.push_back(Pair("sigops", pblocktemplate->vTxSigOps[index_in_template])); // 交易签名操作

        transactions.push_back(entry);
    }

    UniValue aux(UniValue::VOBJ);
    aux.push_back(Pair("flags", HexStr(COINBASE_FLAGS.begin(), COINBASE_FLAGS.end())));

    arith_uint256 hashTarget = arith_uint256().SetCompact(pblock->nBits); // 计算难度目标值

    static UniValue aMutable(UniValue::VARR);
    if (aMutable.empty())
    {
        aMutable.push_back("time"); // 时间
        aMutable.push_back("transactions"); // 交易
        aMutable.push_back("prevblock"); // 前一个区块
    }

    UniValue result(UniValue::VOBJ);
    result.push_back(Pair("capabilities", aCaps)); // 功能
    result.push_back(Pair("version", pblock->nVersion)); // 区块版本
    result.push_back(Pair("previousblockhash", pblock->hashPrevBlock.GetHex())); // 前一个区块哈希
    result.push_back(Pair("transactions", transactions)); // 交易
    result.push_back(Pair("coinbaseaux", aux)); // coinbase aux
    result.push_back(Pair("coinbasevalue", (int64_t)pblock->vtx[0].vout[0].nValue)); // 创币交易输出金额
    result.push_back(Pair("longpollid", chainActive.Tip()->GetBlockHash().GetHex() + i64tostr(nTransactionsUpdatedLast)));
    result.push_back(Pair("target", hashTarget.GetHex())); // 难度目标
    result.push_back(Pair("mintime", (int64_t)pindexPrev->GetMedianTimePast()+1));
    result.push_back(Pair("mutable", aMutable));
    result.push_back(Pair("noncerange", "00000000ffffffff")); // 随机数范围
    result.push_back(Pair("sigoplimit", (int64_t)MAX_BLOCK_SIGOPS)); // 区块签名操作数量上限
    result.push_back(Pair("sizelimit", (int64_t)MAX_BLOCK_SIZE)); // 区块大小上限
    result.push_back(Pair("curtime", pblock->GetBlockTime())); // 区块创建时间
    result.push_back(Pair("bits", strprintf("%08x", pblock->nBits))); // 难度
    result.push_back(Pair("height", (int64_t)(pindexPrev->nHeight+1))); // 高度

    return result; // 返回结果
}
{% endhighlight %}

基本流程：<br>
1.处理命令帮助和参数个数。<br>
2.上锁。<br>
3.获取指定参数。<br>
4.根据参数创建新区块模板。<br>
5.获取区块模板相关信息并返回。

Thanks for your time.

## 参照
* [Developer Documentation - Bitcoin](https://bitcoin.org/en/developer-documentation)
* [Bitcoin Developer Reference - Bitcoin](https://bitcoin.org/en/developer-reference#getblocktemplate)
* [精通比特币（第二版） \| 巴比特图书](http://book.8btc.com/masterbitcoin2cn)
* [...](https://github.com/mistydew/blockchain)