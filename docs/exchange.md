# 联合学徒录取与分期出资联签的交换资料

企业和职业院校共同招聘学徒，分别确认培养席位、补助份额和入职登记。

金额为人民币分，在各月分别适用。null 表示对学校不限。slot_funding 指常规独立资金来源，reserve 资金在确认替代后才可用。队列中的位置按给出的面谈位次排列。
joint-sign 的 target 为培养席位，terms 是确认的起始日和各月待遇，shares 是拟签发时各方按月承担的金额。signed 表示所有签方已经确认；未核实到 signed 时，回执只证明受理。withdrawal 相关 revocation_receipt 是资助方针对未完成联签的处理，不改写已联签待遇。onboard 引用 joint-sign 请求号，enrolled 表示入职方已经登记。

已开通的来源流为 `sponsors`、`countersign`。除 target 外，各操作的报文字段如下。

- `joint-sign` 的提交字段为 `applicant`、`slot`、`terms`、`shares`。
- `onboard` 的提交字段为 `applicant`、`slot`、`joint_request`。

## 本地联调端

地址默认是 `http://127.0.0.1:8091`。交换格式为 UTF-8 JSON。来源编号的作用域由上面的业务约定定义。各来源的 `sequence` 只表示该来源的送达次序，从 1 开始，和业务时刻无关。

- `GET /feeds/{stream}?after=0&limit=20` 返回当前可见的 `items`、`cursor` 和是否还有可见页的 `has_more`。`after` 为上次已收到的序号，`limit` 可取 1 至 100。每一条有 `kind` 和 `data`。按同一位置重读可以看到原报文。
- `POST /operations/{action}/{request_id}` 提交该操作。JSON 中带 `target` 及该操作约定的字段。请求号在同一操作内唯一，重送相同随附值不会执行第二次，改变报文则返回 409。请求号和来源名采用 URL 单段，避免斜杠。
- `GET /operations/{action}/{request_id}` 查询回执，正面回执有 `action`、`request_id`、`target` 和 `status`。提交返回 503 时，外方可能已受理。查询暂时返回 404 只有“查不到证明”的含义，业务最终拒绝以相应明确回执为据。
- `POST /lab/advance` 接受包含正整数 `steps` 的对象。它只推进联调端送达阶段，不修改任何业务日期。阶段从 0 开始。清空联调应另用一个数据库文件，保留该文件重启可继续查询既有回执。

`materials/partner.json` 内的 `feeds` 记录可供演练的来源报文，`visible_at` 是联调的最早送达阶段。`actions` 记录提交字段与外方回执。各操作可以为特定 `target` 单独设定延迟等传输条件。目录内资料均为合成内容。
