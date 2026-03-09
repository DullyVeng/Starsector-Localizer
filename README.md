# Starsector Localizer

一个适配 Codex CLI 的《远行星号》模组汉化 skill，用于把 Starsector 模组中的静态文本和部分 Java/JAR 硬编码提示安全地翻译成中文。

这个 skill 的重点不是“只会翻 CSV”，而是把整套实际可落地的汉化流程固化下来，包括：

- `CSV / JSON / .faction / descriptions` 等静态文本汉化
- Java tooltip、hullmod、shipsystem 等硬编码文本定位
- `JAR` 备份、定向编译、回写 class
- Java 版本目标控制，避免出现 `UnsupportedClassVersionError`
- 编译前英文残留扫描，尽量一次性收完整个 tooltip 区块

## 适用场景

适合这类任务：

- 汉化 Starsector 模组的舰船、武器、插件、势力、描述文本
- 检查某个插件说明为什么没有汉化
- 判断某段文本是在 `CSV` 里还是硬编码在 `Java/JAR` 里
- 修改源码后重新编译并打回模组 `jar`
- 避免 Janino、编码、class version、乱码 bullet 这些常见坑

## Skill 内容

仓库当前包含：

- [SKILL.md](./SKILL.md)
  - 主 skill 定义
  - 汉化流程、风险边界、tooltip 扫描规则
- [agents/openai.yaml](./agents/openai.yaml)
  - Codex skill 元数据
- [references/localization-workflow.md](./references/localization-workflow.md)
  - Starsector 本地化 SOP
  - JAR 编译/回写流程
  - Java 版本与编译防坑清单

## 解决的问题

这个 skill 主要解决下面几类重复调试问题：

1. 改了 `.java` 但游戏实际加载的是 `jar` 里的 `.class`
2. 用主机默认 JDK 编译后，游戏报 `Compiled for the wrong version of Java`
3. 只翻了主 tooltip，遗漏了“系统改装 / 杂项改装”里分散在 helper 方法中的说明
4. `setBulletedListMode` 里混入坏字符，游戏里显示成 `??`
5. `javac` 意外重编整个依赖树，导致依赖库缺失或报一串无关错误

## 使用方式

把这个 skill 放进 Codex 的 skills 目录后，在处理 Starsector 模组汉化时触发它即可。

推荐任务表达方式：

```text
请根据 Starsector Localizer 的流程，对 mods/<mod-name> 进行汉化。
先检查 data 下的核心文件，再判断未汉化文本是在静态文件还是 Java/JAR 中。
如果需要改 jar，请按 skill 里的流程完成备份、编译、回写和版本校验。
```

## 汉化效果

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/31ccce3d-cb08-4ee3-a88f-36a551ab9e40" />
