# Fonoster：Twilio 的开源替代方案

[Fonoster](https://fonoster.com) 正在研究一个创新的可编程电信堆栈，它将允许企业完全通过基于云的实用程序将电话服务与互联网连接。

<a href="https://discord.gg/mpWSRUhG7e"><img alt="Fonoster 社区横幅" src="https://raw.githubusercontent.com/fonoster/.github/main/profile/community.png"></img></a>

![build](https://github.com/fonoster/fonoster/workflows/unit%20tests/badge.svg) [![release](https://github.com/fonoster/fonoster/actions/workflows/release.yaml/badge.svg)](https://github.com/fonoster/fonoster/actions/workflows/release.yaml) [![Discord](https://img.shields.io/discord/1016419835455996076?color=5865F2&label=Discord&logo=discord&logoColor=white)](https://discord.gg/4QWgSz4hTC) <a href="https://github.com/fonoster/fonoster/blob/main/CODE_OF_CONDUCT.md"><img src="https://img.shields.io/badge/Code%20of%20Conduct-v1.0-ff69b4.svg?color=%2347b96d" alt="行为准则"></a> ![GitHub](https://img.shields.io/github/license/fonoster/fonoster?color=%2347b96d) ![Twitter Follow](https://img.shields.io/twitter/follow/fonoster?style=social)

## 功能特性

Fonoster 最显著的功能包括：

- [x] 多租户支持
- [x] 轻松部署 PBX 功能
- [x] 可编程语音应用程序
- [x] NodeJS SDK
- [x] 支持 Amazon 简单存储服务 (S3)
- [x] 使用 Let's Encrypt 的安全 API 端点
- [x] OAuth2 身份验证
- [X] JWT 身份验证
- [x] 基于角色的访问控制 (RBAC)
- [x] 基于插件的命令行工具
- [x] 支持 Google 语音 API

## 代码示例

语音应用程序是控制通话流程的服务器。语音应用程序可以使用以下动词的任意组合：

- `Answer` - 接受来电
- `Hangup` - 关闭通话
- `Play` - 接受包含媒体文件的 URL 并将声音流式传输回呼叫方
- `PlayDtmf` - 接受 DTMF 序列并将其播放回呼叫方
- `Say` - 接受文本，将文本合成为音频，并流式传输回结果
- `Gather` - 等待 DTMF 或语音事件并返回结果
- `SGather` - 返回用于未来 DTMF 和语音结果的流
- `Stream` - 创建双向流以从呼叫者发送和接收音频
- `Dial` - 将通话转接给代理或 PSTN 中的号码
- `Record` - 记录呼叫方的语音并将音频保存在存储子系统上
- `Mute` - 告诉通道停止发送媒体，有效地静音通道
- `Unmute` - 告诉通道允许媒体流

语音应用程序示例：

```typescript
const VoiceServer = require("@fonoster/voice").default;
const { 
  GatherSource, 
  VoiceRequest, 
  VoiceResponse 
} = require("@fonoster/voice");

new VoiceServer().listen(async (req: VoiceRequest, voice: VoiceResponse) => {
  const { ingressNumber, sessionRef, appRef } = req;

  await voice.answer();

  await voice.say("您好！请问您的姓名是什么？");

  const { speech: name } = await voice.gather({
    source: GatherSource.SPEECH
  });

  await voice.say("很高兴认识您 " + name + "！");

  await voice.say("请输入您的 4 位数字密码。");

  const { digits } = await voice.gather({
    maxDigits: 4,
    finishOnKey: "#"
  });

  await voice.say("您的密码是 " + digits);

  await voice.hangup();
});

// 您的应用程序将运行在 tcp://127.0.0.1:50061
// 您可以使用以下命令轻松将其发布到互联网：
// ngrok tcp 50061
```

Fonoster 中的一切都是 API 优先的，发起通话也不例外。您可以使用 SDK 通过几行代码开始通话。

使用 SDK 发起通话的示例：

```typescript
const SDK = require("@fonoster/sdk");

async function main(request) {
  const apiKey = "your-api-key";
  const apiSecret = "your-api-secret"
  const accessKeyId = "WO00000000000000000000000000000000";

  const client = new SDK.Client({ accessKeyId });
  await client.loginWithApiKey(apiKey, apiSecret);

  const calls = new SDK.Calls(client);
  const response = await calls.createCall(request);

  console.log(response); // 成功响应
}

const request = {
  from: "+18287854037",
  to: "+17853178070",
  appRef: "3e61ecb7-a1b6-4a93-84c3-4f1979165bca",
  // 要发送到语音应用程序的可选元数据
  metadata: {
    name: "张三",
    message: "请回电给我。"
  }
};

main(request).catch(console.error);
```

## 入门指南

要开始使用 Fonoster，请使用以下资源：

- [使用 Docker 部署 Fonoster](https://docs.fonoster.com/self-hosting)
- [早期访问用户指南](https://docs.fonoster.com/quickstart)
- [Fonoster 入门指南](https://docs.fonoster.com/quickstart)
- [我们如何创建 Twilio 的开源替代方案以及为什么它很重要](https://dev.to/fonoster/how-we-created-an-open-source-alternative-to-twilio-and-why-it-matters-434g)

## 给个星标！⭐

如果您喜欢这个项目或计划使用它，请给它一个星标。谢谢 🙏

## 错误报告和反馈

对于错误、问题和讨论，请使用 [Github Issues](https://github.com/fonoster/fonoster/issues)

## 贡献

有关贡献，请参阅以下链接：

 - [贡献文档](https://github.com/fonoster/fonoster/blob/main/CONTRIBUTING.md)
 - [贡献者](https://github.com/fonoster/fonoster/contributors)

<!--contributors-->

<table>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/psanders>
            <img src=https://avatars.githubusercontent.com/u/539774?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Pedro Sanders/>
            <br />
            <sub style="font-size:14px"><b>Pedro Sanders</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/efraa>
            <img src=https://avatars.githubusercontent.com/u/40646537?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Efrain Peralta/>
            <br />
            <sub style="font-size:14px"><b>Efrain Peralta</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/angelbencosme>
            <img src=https://avatars.githubusercontent.com/u/6846866?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Angel M. Bencosme/>
            <br />
            <sub style="font-size:14px"><b>Angel M. Bencosme</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/whernandez>
            <img src=https://avatars.githubusercontent.com/u/37089069?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Wandy Hernandez/>
            <br />
            <sub style="font-size:14px"><b>Wandy Hernandez</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/obrucheoghene>
            <img src=https://avatars.githubusercontent.com/u/111436934?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Obruche Wilfred  Oghenechohwo/>
            <br />
            <sub style="font-size:14px"><b>Obruche Wilfred  Oghenechohwo</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/wardner>
            <img src=https://avatars.githubusercontent.com/u/51765669?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Wardner Lara/>
            <br />
            <sub style="font-size:14px"><b>Wardner Lara</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/rihernandez>
            <img src=https://avatars.githubusercontent.com/u/27718122?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Richard HC/>
            <br />
            <sub style="font-size:14px"><b>Richard HC</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/Nageswari-droid>
            <img src=https://avatars.githubusercontent.com/u/65342122?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Nageswari/>
            <br />
            <sub style="font-size:14px"><b>Nageswari</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/xquanluu>
            <img src=https://avatars.githubusercontent.com/u/110280845?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Hoan Luu Huu/>
            <br />
            <sub style="font-size:14px"><b>Hoan Luu Huu</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/speedymonster>
            <img src=https://avatars.githubusercontent.com/u/31810381?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Speedy Monster/>
            <br />
            <sub style="font-size:14px"><b>Speedy Monster</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/parz3val>
            <img src=https://avatars.githubusercontent.com/u/34773307?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=harry_dev/>
            <br />
            <sub style="font-size:14px"><b>harry_dev</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/CKanishka>
            <img src=https://avatars.githubusercontent.com/u/30779692?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Kanishka Chowdhury/>
            <br />
            <sub style="font-size:14px"><b>Kanishka Chowdhury</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/BrayanMnz>
            <img src=https://avatars.githubusercontent.com/u/61812255?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Brayan Munoz V./>
            <br />
            <sub style="font-size:14px"><b>Brayan Munoz V.</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/milis92>
            <img src=https://avatars.githubusercontent.com/u/13440798?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Ivan Milisavljevic />
            <br />
            <sub style="font-size:14px"><b>Ivan Milisavljevic </b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/dedekrnwan>
            <img src=https://avatars.githubusercontent.com/u/25242055?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Dede kurniawan/>
            <br />
            <sub style="font-size:14px"><b>Dede kurniawan</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/gad2103>
            <img src=https://avatars.githubusercontent.com/u/1045265?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=gabriel duncan/>
            <br />
            <sub style="font-size:14px"><b>gabriel duncan</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/iamppborah>
            <img src=https://avatars.githubusercontent.com/u/96339995?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Prasurjya Pran Borah/>
            <br />
            <sub style="font-size:14px"><b>Prasurjya Pran Borah</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/justjordan15>
            <img src=https://avatars.githubusercontent.com/u/164441222?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Jordan/>
            <br />
            <sub style="font-size:14px"><b>Jordan</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/hectorvent>
            <img src=https://avatars.githubusercontent.com/u/2405682?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Hector Ventura/>
            <br />
            <sub style="font-size:14px"><b>Hector Ventura</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/0xflotus>
            <img src=https://avatars.githubusercontent.com/u/26602940?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=0xflotus/>
            <br />
            <sub style="font-size:14px"><b>0xflotus</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/itzmanish>
            <img src=https://avatars.githubusercontent.com/u/12438068?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Manish/>
            <br />
            <sub style="font-size:14px"><b>Manish</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/osehgol>
            <img src=https://avatars.githubusercontent.com/u/4996423?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Osama Sehgol/>
            <br />
            <sub style="font-size:14px"><b>Osama Sehgol</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/psuet>
            <img src=https://avatars.githubusercontent.com/u/7604288?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Paul Sütterlin/>
            <br />
            <sub style="font-size:14px"><b>Paul Sütterlin</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/RiadVargas>
            <img src=https://avatars.githubusercontent.com/u/4274014?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Riad Vargas/>
            <br />
            <sub style="font-size:14px"><b>Riad Vargas</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/vcidst>
            <img src=https://avatars.githubusercontent.com/u/683016?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Shailendra Paliwal/>
            <br />
            <sub style="font-size:14px"><b>Shailendra Paliwal</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/gitter-badger>
            <img src=https://avatars.githubusercontent.com/u/8518239?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=The Gitter Badger/>
            <br />
            <sub style="font-size:14px"><b>The Gitter Badger</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/YuriCodes>
            <img src=https://avatars.githubusercontent.com/u/80093500?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Yuri/>
            <br />
            <sub style="font-size:14px"><b>Yuri</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/cdrsociate>
            <img src=https://avatars.githubusercontent.com/u/89363212?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=cdrsociate/>
            <br />
            <sub style="font-size:14px"><b>cdrsociate</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/ghana7989>
            <img src=https://avatars.githubusercontent.com/u/65382745?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=pavan/>
            <br />
            <sub style="font-size:14px"><b>pavan</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/nrjchnd>
            <img src=https://avatars.githubusercontent.com/u/17134818?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=nrjchnd/>
            <br />
            <sub style="font-size:14px"><b>nrjchnd</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/salami-dev>
            <img src=https://avatars.githubusercontent.com/u/57477131?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Salami Bashir/>
            <br />
            <sub style="font-size:14px"><b>Salami Bashir</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/scshiv29-dev>
            <img src=https://avatars.githubusercontent.com/u/68141773?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Shivam Deepak Chaudhary/>
            <br />
            <sub style="font-size:14px"><b>Shivam Deepak Chaudhary</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/showf68>
            <img src=https://avatars.githubusercontent.com/u/45857918?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Yossef Haim/>
            <br />
            <sub style="font-size:14px"><b>Yossef Haim</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/telenautical>
            <img src=https://avatars.githubusercontent.com/u/106842020?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=telenautical/>
            <br />
            <sub style="font-size:14px"><b>telenautical</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/theewiz>
            <img src=https://avatars.githubusercontent.com/u/81051645?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Wisdom Elendu/>
            <br />
            <sub style="font-size:14px"><b>Wisdom Elendu</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/judgegodwins>
            <img src=https://avatars.githubusercontent.com/u/38760034?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Judge Godwins/>
            <br />
            <sub style="font-size:14px"><b>Judge Godwins</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/jonathan-chin>
            <img src=https://avatars.githubusercontent.com/u/7519412?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Jon Chin/>
            <br />
            <sub style="font-size:14px"><b>Jon Chin</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/harish-chander>
            <img src=https://avatars.githubusercontent.com/u/13236956?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Harish Chander/>
            <br />
            <sub style="font-size:14px"><b>Harish Chander</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/GaryBarnes17>
            <img src=https://avatars.githubusercontent.com/u/97693048?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Gary Barnes/>
            <br />
            <sub style="font-size:14px"><b>Gary Barnes</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/FidalMathew>
            <img src=https://avatars.githubusercontent.com/u/84982038?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Fidal Mathew/>
            <br />
            <sub style="font-size:14px"><b>Fidal Mathew</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/eatskolnikov>
            <img src=https://avatars.githubusercontent.com/u/1693000?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Enmanuel Toribio/>
            <br />
            <sub style="font-size:14px"><b>Enmanuel Toribio</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/jellydn>
            <img src=https://avatars.githubusercontent.com/u/870029?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Dung Duc Huynh (Kaka)/>
            <br />
            <sub style="font-size:14px"><b>Dung Duc Huynh (Kaka)</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/cdosoftei>
            <img src=https://avatars.githubusercontent.com/u/7636091?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Ciprian/>
            <br />
            <sub style="font-size:14px"><b>Ciprian</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/infinitydon>
            <img src=https://avatars.githubusercontent.com/u/6318992?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Christopher Adigun/>
            <br />
            <sub style="font-size:14px"><b>Christopher Adigun</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/brunowego>
            <img src=https://avatars.githubusercontent.com/u/441774?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Bruno Gomes/>
            <br />
            <sub style="font-size:14px"><b>Bruno Gomes</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/brunoarueira>
            <img src=https://avatars.githubusercontent.com/u/119518?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Bruno Arueira/>
            <br />
            <sub style="font-size:14px"><b>Bruno Arueira</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/antoniusostermann>
            <img src=https://avatars.githubusercontent.com/u/2332002?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Antonius Ostermann/>
            <br />
            <sub style="font-size:14px"><b>Antonius Ostermann</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/alifiratari>
            <img src=https://avatars.githubusercontent.com/u/10004438?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Ali Firat ARI/>
            <br />
            <sub style="font-size:14px"><b>Ali Firat ARI</b></sub>
        </a>
    </td>
</tr>
<tr>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/alexsands>
            <img src=https://avatars.githubusercontent.com/u/4269772?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Alex/>
            <br />
            <sub style="font-size:14px"><b>Alex</b></sub>
        </a>
    </td>
    <td align="center" style="word-wrap: break-word; width: 150.0; height: 150.0">
        <a href=https://github.com/itsalb3rt>
            <img src=https://avatars.githubusercontent.com/u/35310226?v=4 width="100;"  style="border-radius:50%;align-items:center;justify-content:center;overflow:hidden;padding-top:10px" alt=Albert E. Hidalgo Taveras/>
            <br />
            <sub style="font-size:14px"><b>Albert E. Hidalgo Taveras</b></sub>
        </a>
    </td>
</tr>
</table>

## 赞助商

我们很高兴得到来自多个行业的受人尊敬的公司和个人的支持。

在[这里](https://github.com/sponsors/fonoster)找到我们所有的支持者

> [成为 Github 赞助商](https://github.com/sponsors/fonoster)

## 作者

 - [Pedro Sanders](https://github.com/psanders)

## 许可证

版权所有 (C) 2025 [Fonoster Inc](https://fonoster.com)。MIT 许可证（详情请参阅 [LICENSE](https://github.com/fonoster/fonoster/blob/main/LICENSE)）。