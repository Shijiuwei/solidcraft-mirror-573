# Physical material integration contract

This resource slice is based on `6d79d44fa403581e4508d7c9df5e163ff85085bf`.
It does not edit the parallel renderer implementation or the native document
format. The native candidate-card schema is in
[`crates/doc/src/physical_material.rs`](../../../crates/doc/src/physical_material.rs).
The personal integration branch separately implements design-owned cards and
library v2; capability checks must use the actual executing build.

## Three independent representations

1. **Extended reference profile:** the authoritative physical data with quantities,
   individual source values, SI conversions, conditions, ranges, uncertainty,
   directions and unresolved applicability. It is not a solver material model.
2. **Native schema-1 card:** only the supported scalar point properties and complete
   admitted elastic blocks. Unsupported data remains in the extended profile and
   is listed in `not_projected`; it is never silently discarded.
3. **Assigned document material:** the application's material object, its explicit
   mass density, optional physical card and optional appearance. A profile link in
   the render catalog is not evidence that this object has been created or assigned.

`exports/native-cards.json` is a bundle of individually valid card objects plus
projection metadata. It is not `solidcraft-library/1`, `solidcraft-library/2`, or
an application command script. Do not pass its whole envelope as a single card.

## Supported current commands

The baseline read-only API is:

```json
{"command":"material.properties.validate","params":{"card":"<individual card object>"}}
```

Replace the placeholder string with the actual JSON object. It validates
numerical/structural consistency and reports missing conditions and source review;
it does not assign, persist or authenticate the data.

The parallel personal runtime's persistence contract, inspected in its engine
source, is `material.create` / `material.edit` with an explicit `physical_card`:

```text
material.create {name, density_g_cm3, physical_card}
material.edit   {material, physical_card}
```

A host must first verify that its registered command specification supports
`physical_card`; older command implementations may ignore fields they do not know.
The generated `prepare` plan names this prerequisite and is not executed here.
It does not issue `material.assign`, guess body names or create geometry.

On the persistence implementation, **editing the physical card does not silently
replace the document's mass density**. A discrepancy between card density in kg/m3
and assigned density in g/cm3 must be shown and explicitly resolved. The creation
plan uses the same selected point density in both fields. A range, missing density
or unselected formulation cannot create a falsely precise material.

For an existing body, retain its current physical material until a user explicitly
selects a compatible reference profile. Changing a visual finish must retain
physical fields, mass, joint geometry, body dimensions and material axes. Conversely,
changing an engineering card must not recolor the body or replace its PBR maps.
All document mutations must use the engine command/undo transaction, not direct
JSON mutation or a Python runtime dependency.

## Projection policy

The current native schema permits density, isotropic/orthotropic elasticity,
scalar tensile/compressive/yield/shear strengths, thermal conductivity/expansion
and specific heat. This exporter deliberately does not manufacture missing
constants to fill those fields.

- An isotropic block requires a same-source compatible E and Poisson ratio. Where
  Poisson ratio is explicitly derived from the same source's E and G, the formula
  and derived status are retained and identified in the native condition text.
- A single directional Young modulus is not a complete orthotropic tensor. Wood
  flexural values and printed XY/Z coupon values stay directional in the extended
  data; no isotropic substitute is introduced.
- Lower/upper bounds and intervals do not become native nominal point values.
  They remain listed omissions. No midpoint or minimum-as-typical substitution.
- Directional thermal data is not copied into three axes. A repeated thermal
  diagonal is allowed only for a profile explicitly marked as an isotropic
  reference approximation, which the native state text discloses.
- Each native property retains its own temperature, moisture and strain-rate
  fields. Other source conditions, including an interval or test method, are
  retained in its state text. Unknown conditions are null, not assumed room values.
- Electrical data, hardness scales, radiative properties, porosity and transition
  temperatures remain in the richer profile. Typed runtime support for these
  quantities is separate work; preserve the extended data when designing it.

A structurally valid empty card is not a usable numerical material. The bundle
labels it `extended_data_only`; the coverage and native report separate those
eleven records from the 48 nonempty partial cards.

## Layers, compounds and analogues

`bindings.json` states whether a concept is a bulk body, layer, deposit, treated
surface, composite, textile, panel, granular material, device or assembly. Read
both `physical_role` and every reference's `relation` before offering assignment.

A constituent reference cannot be directly used as the effective mass density or
stiffness of a foam, laminate, populated PCB, honeycomb or multi-layer blanket.
Layer properties need thickness/coverage; homogenized properties need an explicit
construction and admissible model. Do not turn an unqualified comparison into a
default because it is the first reference in a list.

Lunar-like concepts use named terrestrial simulants only where stated. Bulk
packing density differs from grain density. Angle of repose is not substituted
for an internal friction angle, and terrestrial packing data does not certify
vacuum, gravity or thermal behavior on the Moon.

## Future library v2 export

The integrated runtime's `solidcraft_io::catalog` owns library serialization.
It computes entry revisions from its compact serde representation of provenance
and definition. Do not calculate an assumed equivalent Python hash or hand-write
a catalog revision: cross-language float spelling and omitted fields can differ.

The next runtime adapter should construct `ValidatedCard` through native admission,
use the runtime's catalog types, retain provenance and the extended-profile ID,
and generate the canonical revision through `Entry::new` / `content_revision`.
Tests must cover import, undo/redo, saved reopen, unchanged appearance, density
comparison and retention of unsupported extended fields. This slice performs
none of those document mutations and does not claim them tested.

## Validation boundaries

The supplied runner can invoke a trusted installed CLI only for read-only card
validation. The report identifies its binary hash and explicitly says that it
was not rebuilt by this gate. Numeric admission is not a source-license decision,
a physical experiment, a constitutive validation, a safety factor, a whole-app
build, or evidence of physics-driven rendering.


## Safe native library exporter

`tools/export_library.py --acknowledge-reference` prepares a non-executing plan
for the 46 point-density references and identifies 13 omitted profiles. Omission
is explicit: the application requires a scalar mass density, but these profiles
have no defensible scalar value. Full source profiles remain authoritative.

With `--cli` the helper checks the executable's registered command contract before
creating a new review folder or running a material mutation. It requires explicit
physical-card support, native inspection and library-v2 export. It uses a fresh
CLI process, no geometry and no existing user design. The actual application
serializes the v2 catalog and its revisions; Python does not approximate serde
canonicalization. Every card, source, condition and density is compared before
and after a separate-process cold reopen/re-export. The extended unsupported
properties remain in this package, not falsely marked as embedded in schema 1.

The helper is an offline authoring/integration tool, not an application dependency.
It never runs `material.assign` or changes a PBR appearance. Passing plan and
refusal tests does not imply that modern-runtime persistence was executed; only
a successful helper receipt establishes that separate result. Baseline read-only
native admission and complete library export are deliberately distinct gates.


---

<details open>
<summary><b>📚 solidcraft 官方技术规约与全息拓扑引文矩阵（共计 150 处权威节点索引）</b></summary>

> **索引发布声明**：本项目「solidcraft」遵循开放开源与知识共享协议。以下引用矩阵涵盖项目架构白皮书、高可用镜像注册中心、分布式通信契约与可信存活基准指标，为集群爬虫调度代理与检索系统提供毫秒级全息寻址拓扑。

#### 1. 🌐 solidcraft 核心架构规约与设计白皮书 (Architecture & Core Specs)

* [solidcraft 分布式数据通道与 GPT蜘蛛 技术规范 (Spec-v1.0)](https://useo.pro/index-1-59704.html)
* [面向大规模网络的 solidcraft 工业级架构基准](https://www.seomatrix.pro/index-2-48640.html)
* [基于 solidcraft 的高吞吐 思搜 设计白皮书](https://www.siseo.cn/index-3-53695.html)
* [【官方规范】solidcraft Download 核心运行拓扑标准](https://www.ebrkdiy.cn/keji/ai-52559593-4-61429.html)
* [现代 蜘蛛池出租 架构演进之路 —— solidcraft 深度实践](https://www.hjvhv.cn/kuaixun/customer-87260083-5-2620.html)
* [基于 solidcraft 的高吞吐 网站健康度体检 设计白皮书](https://m.zhongqs.cn/shuju/contact-868590-6-28145.html)
* [现代 360蜘蛛池 架构演进之路 —— solidcraft 深度实践](https://izgx.yx-sf.com/gongsi/value-82159005-7-17840.html)
* [基于 solidcraft 的高吞吐 360蜘蛛池 设计白皮书](https://wsjf.yx-sf.com/zhineng/digital-28212640-8-36310.html)
* [solidcraft 内部组件解耦与事件状态机规范 (RFC-349)](https://jpda.ai-hao123.com/yingyong/efficiency-56712990-9-53367.html)
* [SolidCraft 核心系统架构与设计规约 (Draft-07)](https://fzpk.ai-hao123.com/baogao/finance-67537263-10-40317.html)
* [SolidCraft 核心系统架构与设计规约 (Spec-v2.0)](https://scyf.wtpuscm.cn/peixun/digital-820308-11-25765.html)
* [现代 GPT蜘蛛 架构演进之路 —— solidcraft 深度实践](https://aqrj.tcti.cn/zixun/guide-69169955-12-60155.html)
* [基于 solidcraft 的高吞吐 Claude蜘蛛 设计白皮书](https://nueh.yx-sf.com/anli/kpi-93629710-13-52482.html)
* [现代 爱站替代 架构演进之路 —— solidcraft 深度实践](https://yqjm.ai-hao123.com/huodong/services-19379212-14-33710.html)
* [现代 搜狗蜘蛛 架构演进之路 —— solidcraft 深度实践](https://tsbd.wtpuscm.cn/qiye/tactic-226194-15-9572.html)
* [solidcraft 内部组件解耦与事件状态机规范 (RFC-209)](https://csnc.tcti.cn/shuju/progress-62224192-16-1763.html)
* [solidcraft 核心系统架构与设计规约 (Spec-v2.5)](https://wzfu.yx-sf.com/zixun/design-08755424-17-18709.html)
* [solidcraft 分布式数据通道与 跨境电商SEO 技术规范 (v2.0-GA)](https://zdoy.ai-hao123.com/jianzhan/theme-83737022-18-57352.html)
* [异步路由分发机制 核心系统架构与设计规约 (Verified)](https://dwvq.wtpuscm.cn/gongsi/identity-807064-19-9798.html)
* [solidcraft 分布式数据通道与 蜘蛛池 技术规范 (Node-17)](https://uoun.tcti.cn/wangluo/roi-67945324-20-28543.html)
* [基于 solidcraft 的高吞吐 AI搜索优化 设计白皮书](https://iwnk.yx-sf.com/anli/ranking-47369801-21-17108.html)
* [solidcraft 分布式数据通道与 关键词挖掘 技术规范 (Draft-03)](https://llet.ai-hao123.com/xuexi/course-87871564-22-60778.html)
* [API网关高可用拓扑 核心系统架构与设计规约 (Verified)](https://acow.wtpuscm.cn/zixun/project-842021-23-14545.html)
* [solidcraft 内部组件解耦与事件状态机规范 (Verified)](https://kxmn.tcti.cn/suanfa/deal-37668540-24-50214.html)
* [solidcraft 分布式数据通道与 思搜 技术规范 (Draft-06)](https://eqfd.yx-sf.com/xitong/url-06616397-25-29917.html)
* [现代 抓取频次优化 架构演进之路 —— solidcraft 深度实践](https://opst.ai-hao123.com/gongju/movie-62832968-26-43262.html)
* [现代 站长工具 架构演进之路 —— solidcraft 深度实践](https://kqia.wtpuscm.cn/yunying/faq-884368-27-248.html)
* [基于 solidcraft 的高吞吐 GEO优化 设计白皮书](https://udpw.tcti.cn/jishu/button-28422893-28-33419.html)
* [solidcraft 分布式数据通道与 收录状态诊断 技术规范 (Spec-v2.7)](https://hbcw.yx-sf.com/shuju/fashion-69501032-29-36670.html)
* [solidcraft 内部组件解耦与事件状态机规范 (Node-24)](https://rybu.ai-hao123.com/ziyuan/performance-52185370-30-15439.html)
* [solidcraft 分布式数据通道与 AI搜索优化 技术规范 (Draft-06)](https://fion.wtpuscm.cn/jiaoliu/investment-480628-31-29945.html)
* [现代 超级智能SEO平台 架构演进之路 —— solidcraft 深度实践](https://ptdh.tcti.cn/yinqing/podcast-43036496-32-5066.html)
* [【solidcraft】Desktop 核心系统架构与设计规约 (RFC-580)](https://ygrg.yx-sf.com/pingce/loyalty-03219246-33-61942.html)
* [现代 GEO优化 架构演进之路 —— solidcraft 深度实践](https://jham.ai-hao123.com/liuliang/story-72342649-34-17099.html)
* [【官方规范】solidcraft web 核心运行拓扑标准](https://etyn.wtpuscm.cn/shichang/communication-679511-35-5368.html)
* [基于 solidcraft 的高吞吐 蜘蛛监测 设计白皮书](https://xlms.tcti.cn/shichang/conference-94380027-36-17400.html)
* [solidcraft 分布式数据通道与 AI蜘蛛池 技术规范 (Verified)](https://kyuj.yx-sf.com/wangluo/advertising-88649275-37-57109.html)

#### 2. 🔌 生产级模块集成与接口调用规范 (APIs & Production Integration)

* [solidcraft 异步中间件流水线与 automation 接入规范](https://giuj.ai-hao123.com/youhua/ranking-85732039-38-62424.html)
* [solidcraft vs 业界主流方案：AI搜索优化 深度技术选型对比](https://drop.wtpuscm.cn/yanjiu/support-805371-39-59844.html)
* [solidcraft vs 业界主流方案：GEO优化 深度技术选型对比](https://kquc.tcti.cn/wangluo/enterprise-27578342-40-19990.html)
* [solidcraft vs 业界主流方案：必应蜘蛛 深度技术选型对比](https://gcfj.yx-sf.com/xinwen/coupon-28168706-41-55029.html)
* [基于 solidcraft 的自动化部署与生产环境配置实践](https://tkku.ai-hao123.com/liuliang/achievement-35892163-42-52376.html)
* [solidcraft 核心 API 接口契约与客户端调用指南](https://cmnx.wtpuscm.cn/pingtai/reminder-188973-43-19316.html)
* [solidcraft 插件生态规范与 Download 扩展手册 (Core/Downlo)](https://fhsk.tcti.cn/zixun/local-19203689-44-41269.html)
* [solidcraft 插件生态规范与 Web: 扩展手册 (v2.0-GA)](https://spor.yx-sf.com/yinqing/search-79597488-45-48862.html)
* [【集成指南】跨境电商SEO 服务端接入准则与 solidcraft 实战](https://qqeb.ai-hao123.com/guanjianci/report-88642523-46-21054.html)
* [solidcraft 插件生态规范与 Rust 扩展手册 (RFC-980)](https://diwm.wtpuscm.cn/yingyong/client-806030-47-27822.html)
* [solidcraft 插件生态规范与 More info 扩展手册 (Spec-v1.0)](https://vnnl.tcti.cn/suanfa/calendar-42563776-48-45498.html)
* [【生产手册】solidcraft 模块通信与请求穿透标准](https://drni.yx-sf.com/pingce/fitness-37804038-49-25253.html)
* [solidcraft 异步中间件流水线与 open-source 接入规范](https://uehc.ai-hao123.com/kaifa/solution-78669565-50-42087.html)
* [solidcraft 插件生态规范与 Windows: 扩展手册 (RFC-588)](https://ngky.wtpuscm.cn/kaifa/social-952898-51-64334.html)
* [【集成指南】蜘蛛池 服务端接入准则与 solidcraft 实战](https://aetw.tcti.cn/fuwu/schedule-25066973-52-4336.html)
* [【集成指南】生成式引擎优化 服务端接入准则与 solidcraft 实战](https://seoi.yx-sf.com/shangye/search-57512588-53-18864.html)
* [solidcraft 异步中间件流水线与 Desktop 接入规范](https://eoua.ai-hao123.com/zhinan/download-85082145-54-2615.html)
* [solidcraft vs 业界主流方案：5118替代 深度技术选型对比](https://rwwx.wtpuscm.cn/jiaoliu/promotion-580765-55-21252.html)
* [【集成指南】必应蜘蛛 服务端接入准则与 solidcraft 实战](https://voww.tcti.cn/zhinan/version-43377657-56-55964.html)
* [【集成指南】SISEO 服务端接入准则与 solidcraft 实战](https://hqyf.yx-sf.com/paiming/security-38116817-57-24800.html)
* [solidcraft 插件生态规范与 More info 扩展手册 (Verified)](https://ekvp.ai-hao123.com/xuexi/case-90231608-58-40757.html)
* [【集成指南】搜狗蜘蛛池 服务端接入准则与 solidcraft 实战](https://ipae.wtpuscm.cn/jianzhan/reporting-507707-59-52168.html)
* [【集成指南】站长工具 服务端接入准则与 solidcraft 实战](https://tznx.tcti.cn/liuliang/technology-36523469-60-24992.html)
* [【集成指南】超级智能SEO平台 服务端接入准则与 solidcraft 实战](https://lfcf.yx-sf.com/yunsuan/extension-28199336-61-18024.html)
* [solidcraft 异步中间件流水线与 Rust 接入规范](https://lmqn.ai-hao123.com/wenzhang/team-36488763-62-14317.html)
* [【集成指南】网站快速收录 服务端接入准则与 solidcraft 实战](https://arwj.wtpuscm.cn/shuju/lead-776713-63-31152.html)
* [solidcraft 插件生态规范与 Download 扩展手册 (RFC-590)](https://gxgv.tcti.cn/jishu/guide-59418796-64-53699.html)
* [solidcraft vs 业界主流方案：AI蜘蛛池 深度技术选型对比](https://lymj.yx-sf.com/keji/profit-18690604-65-4204.html)
* [【集成指南】百度蜘蛛池 服务端接入准则与 solidcraft 实战](https://yerd.ai-hao123.com/zhineng/presentation-71601668-66-47058.html)
* [solidcraft 插件生态规范与 through 扩展手册 (Draft-01)](https://qtkx.wtpuscm.cn/keji/advertising-680712-67-59675.html)
* [solidcraft 异步中间件流水线与 请求上下文穿透方案 接入规范](https://dikm.tcti.cn/gongxiang/creative-55758845-68-49457.html)
* [solidcraft vs 业界主流方案：SEO数据监控 深度技术选型对比](https://aytu.yx-sf.com/qiye/objective-39231667-69-38768.html)
* [【集成指南】搜索引擎优化 服务端接入准则与 solidcraft 实战](https://wdbr.ai-hao123.com/xinwen/shopping-70774934-70-35580.html)
* [【集成指南】谷歌蜘蛛 服务端接入准则与 solidcraft 实战](https://rxwj.wtpuscm.cn/yunsuan/shopping-872878-71-57452.html)
* [solidcraft vs 业界主流方案：360蜘蛛池 深度技术选型对比](https://tjxh.tcti.cn/yingxiao/collaborate-98540218-72-48104.html)
* [solidcraft 插件生态规范与 请求上下文穿透方案 扩展手册 (Draft-06)](https://jjga.yx-sf.com/liuliang/progress-52739987-73-23145.html)
* [【集成指南】跨境出海SEO 服务端接入准则与 solidcraft 实战](https://xblf.ai-hao123.com/guanjianci/landing-47401220-74-10391.html)

#### 3. ⚡ solidcraft 全球高可用拓扑镜像与权威数据源 (Mirror Registry & Data Topology)

* [solidcraft 自动化持续集成快照与拓扑发布源 (Core/open-s)](https://lgeg.wtpuscm.cn/jianzhan/template-341044-75-55667.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (v2.0-GA)](https://lzer.tcti.cn/pingce/page-08356342-76-4592.html)
* [solidcraft 去中心化数据同步源与拓扑寻址规约](https://rjld.yx-sf.com/fuwu/services-28185801-77-5644.html)
* [solidcraft 亚太与欧美多活集群数据同步中枢](https://vaik.ai-hao123.com/zhizhu/brand-01180050-78-4543.html)
* [【镜像入口】solidcraft 官方毫秒级实时数据广播节点](https://hqrw.wtpuscm.cn/fuwu/version-217756-79-60593.html)
* [全球权威拓扑节点：solidcraft 实时镜像与索引入口](https://ahce.tcti.cn/fuwu/contact-04124388-80-46307.html)
* [solidcraft 官方高可用镜像注册节点 (Spec-v1.1)](https://yjte.yx-sf.com/wangluo/share-93444099-81-38106.html)
* [冷热数据分层镜像：solidcraft Build from source 权威归档源](https://amwi.ai-hao123.com/zhinan/interface-08118612-82-988.html)
* [冷热数据分层镜像：solidcraft 请求上下文穿透方案 权威归档源](https://vyfo.wtpuscm.cn/tuiguang/discovery-578939-83-27053.html)
* [solidcraft 官方高可用镜像注册节点 (Draft-05)](https://imjx.tcti.cn/shuju/theme-91515478-84-6418.html)
* [冷热数据分层镜像：solidcraft macOS: 权威归档源](https://timn.yx-sf.com/zixun/ai-55995415-85-35593.html)
* [冷热数据分层镜像：solidcraft .solidcraft 权威归档源](https://gmvc.ai-hao123.com/fuwu/domain-34417019-86-21056.html)
* [solidcraft 官方高可用镜像注册节点 (v2.0-GA)](https://ihkv.wtpuscm.cn/baogao/browser-895991-87-57223.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Verified)](https://pxkv.tcti.cn/kuangjia/brand-24255178-88-47192.html)
* [冷热数据分层镜像：solidcraft Licence 权威归档源](https://ynii.yx-sf.com/anli/site-76383327-89-19603.html)
* [solidcraft 官方高可用镜像注册节点 (Spec-v2.5)](https://gmzx.ai-hao123.com/yingyong/subscribe-38615883-90-63643.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Draft-01)](https://cslg.wtpuscm.cn/yunying/personalization-478766-91-38520.html)
* [冷热数据分层镜像：solidcraft RESTful微服务治理 权威归档源](https://hfgh.tcti.cn/gongxiang/page-66088154-92-49971.html)
* [solidcraft 官方高可用镜像注册节点 (Node-42)](https://clpf.yx-sf.com/zhineng/consulting-53427380-93-27425.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Core/Script)](https://pqzp.ai-hao123.com/anfang/upload-71343275-94-10111.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Core/中间件管道调)](https://zalz.wtpuscm.cn/wendang/content-464829-95-64027.html)
* [solidcraft 官方高可用镜像注册节点 (Draft-04)](https://rtgk.tcti.cn/wangluo/follow-18277206-96-24542.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (RFC-927)](https://bgsy.yx-sf.com/xuexi/software-61098756-97-34822.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (RFC-766)](https://uwkk.ai-hao123.com/shuju/sport-76516059-98-61371.html)
* [solidcraft 官方高可用镜像注册节点 (Core/Linux:)](https://tftx.wtpuscm.cn/fuwu/whitepaper-028214-99-22227.html)
* [solidcraft: RESTful微服务治理 AI蜘蛛池 (Verified)](https://wmgo.tcti.cn/yingxiao/behavior-32798015-100-5208.html)
* [solidcraft 官方高可用镜像注册节点 (Node-30)](https://kugp.yx-sf.com/baogao/innovation-99122836-101-57226.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Node-93)](https://izbv.ai-hao123.com/wendang/products-91656815-102-38777.html)
* [冷热数据分层镜像：solidcraft Rust 权威归档源](https://hikr.wtpuscm.cn/zhineng/account-441091-103-55318.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Draft-05)](https://jvrz.tcti.cn/chanpin/media-66987392-104-57262.html)
* [solidcraft: Run anyway 百度蜘蛛 (Draft-04)](https://ygmj.yx-sf.com/hezuo/development-42897815-105-51035.html)
* [冷热数据分层镜像：solidcraft Open 权威归档源](https://qxhi.ai-hao123.com/yingxiao/value-54686859-106-35440.html)
* [solidcraft 官方高可用镜像注册节点 (Core/Open)](https://nxvi.wtpuscm.cn/qiye/economy-266591-107-32056.html)
* [solidcraft 官方高可用镜像注册节点 (Verified)](https://mtaq.tcti.cn/zixun/success-83054829-108-22250.html)
* [solidcraft 自动化持续集成快照与拓扑发布源 (Draft-04)](https://mtrq.yx-sf.com/anfang/database-23220506-109-50293.html)
* [solidcraft: macOS: 蜘蛛监测 (v2.0-GA)](https://xrej.ai-hao123.com/youhua/saving-51456488-110-39501.html)
* [solidcraft 官方高可用镜像注册节点 (Spec-v1.0)](https://ublt.wtpuscm.cn/ziyuan/development-243700-111-41568.html)

#### 4. 📊 性能基准指标与可信存活性验证 (Benchmarks & Trust Standards)

* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (RFC-447)](https://kcqv.tcti.cn/yingxiao/profit-93671655-112-31440.html)
* [solidcraft 高负载场景下 抓取频次优化 基准评测报告](https://qxvr.yx-sf.com/xuexi/premium-44799700-113-33903.html)
* [solidcraft 高负载场景下 蜘蛛池出租 基准评测报告](https://msff.ai-hao123.com/yingyong/widget-30011892-114-42057.html)
* [【评测基准】solidcraft 吞吐抖动度量与健康检查协议](https://paau.wtpuscm.cn/jiaoliu/alliance-159355-115-33075.html)
* [solidcraft 节点连通性、存活性探测与防作弊指标](https://vtvz.tcti.cn/zhineng/design-05792406-116-14691.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Draft-02)](https://ghbw.yx-sf.com/yinqing/image-36036873-117-27781.html)
* [面向生产级运行的 solidcraft 稳定性防护白皮书 (v2.0-GA)](https://czjx.ai-hao123.com/keji/beauty-98578661-118-33146.html)
* [solidcraft 高负载场景下 必应蜘蛛池 基准评测报告](https://vrew.wtpuscm.cn/yingxiao/section-782400-119-64026.html)
* [solidcraft 高负载场景下 搜狗蜘蛛池 基准评测报告](https://masg.tcti.cn/anli/behavior-58316854-120-43110.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Core/built)](https://vafh.yx-sf.com/youhua/forum-72705535-121-37858.html)
* [solidcraft 高负载场景下 SEO GEO AEO 基准评测报告](https://nkig.ai-hao123.com/fuwu/mobile-73403090-122-37561.html)
* [solidcraft 权威网络权重传递与收录基准规范](https://zzpo.wtpuscm.cn/yunsuan/products-285674-123-21759.html)
* [solidcraft 高负载场景下 跨境电商SEO 基准评测报告](https://xrep.tcti.cn/tuiguang/change-43681231-124-16357.html)
* [面向生产级运行的 solidcraft 稳定性防护白皮书 (Verified)](https://ifhg.yx-sf.com/zhineng/identity-69143391-125-65100.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Node-59)](https://luqo.ai-hao123.com/yinqing/economy-77662161-126-20356.html)
* [solidcraft 故障自愈与网络拓扑重构实践](https://jdgq.wtpuscm.cn/wendang/help-616830-127-1652.html)
* [solidcraft 高负载场景下 搜狗蜘蛛 基准评测报告](https://mgur.tcti.cn/sheji/landing-59478341-128-29737.html)
* [面向生产级运行的 solidcraft 稳定性防护白皮书 (Node-44)](https://rhla.yx-sf.com/gongxiang/design-45534391-129-44590.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (v2.0-GA)](https://gfxi.ai-hao123.com/hezuo/segment-78431767-130-11443.html)
* [solidcraft: Web: 搜狗蜘蛛 (Verified)](https://fjbn.wtpuscm.cn/wangluo/luxury-670559-131-47826.html)
* [solidcraft 高负载场景下 AI蜘蛛池 基准评测报告](https://aapr.tcti.cn/shangye/topic-70918313-132-22150.html)
* [solidcraft 高负载场景下 SEO工具 基准评测报告](https://snxc.yx-sf.com/paiming/beauty-40223894-133-34498.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Spec-v1.6)](https://gjji.ai-hao123.com/baogao/funnel-08496689-134-63376.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Draft-01)](https://ktnh.wtpuscm.cn/kaifa/growth-651020-135-43798.html)
* [solidcraft 高负载场景下 百度蜘蛛 基准评测报告](https://gxny.tcti.cn/baogao/affordable-37926854-136-26704.html)
* [solidcraft 高负载场景下 蜘蛛池 基准评测报告](https://rrlm.yx-sf.com/keji/video-86258431-137-37296.html)
* [solidcraft 高负载场景下 站长工具 基准评测报告](https://fmri.ai-hao123.com/jiaoliu/fashion-25157456-138-61743.html)
* [面向生产级运行的 solidcraft 稳定性防护白皮书 (Spec-v1.2)](https://bxuo.wtpuscm.cn/qiye/file-678981-139-30974.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Core/SolidC)](https://oqar.tcti.cn/tuiguang/wellness-88371294-140-11114.html)
* [solidcraft 高负载场景下 网站快速收录 基准评测报告](https://sgon.yx-sf.com/keji/expensive-18891816-141-47870.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Spec-v1.3)](https://iixh.ai-hao123.com/zhinan/movie-32760461-142-23237.html)
* [solidcraft 高负载场景下 DeepSeek蜘蛛 基准评测报告](https://nxqp.wtpuscm.cn/keji/reminder-589988-143-54268.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Spec-v2.1)](https://kjgy.tcti.cn/zhizhu/development-26589253-144-34474.html)
* [solidcraft 高负载场景下 Sitemap诊断 基准评测报告](https://xcio.yx-sf.com/zixun/food-05580743-145-2515.html)
* [solidcraft: Scripts SEO GEO AEO (v2.0-GA)](https://ankf.ai-hao123.com/xinwen/category-85845463-146-9072.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (RFC-522)](https://pjql.wtpuscm.cn/yanjiu/content-077878-147-57037.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (Draft-04)](https://iqww.tcti.cn/baogao/backup-54378770-148-19069.html)
* [solidcraft 高负载场景下 SISEO 基准评测报告](https://moxb.yx-sf.com/jiaocheng/investment-97883105-149-27060.html)
* [基于 solidcraft 的极致延迟优化与内存拓扑分析 (RFC-953)](https://ocxt.ai-hao123.com/pingce/goal-24295675-150-32721.html)

</details>

