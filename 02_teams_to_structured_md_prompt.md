# Teams設計検討会 → 構造化Markdown生成プロンプト

## 1. 目的

このプロンプトは、Teamsの設計検討会Transcriptから、単純な議事録要約ではなく、**設計者の判断・経験・ノウハウが含まれる議論構造をIssue単位で抽出する**ために使用する。

最重要方針は以下。

- 採用された結論だけを残さない
- 却下された案も残す
- なぜ良い/悪いと評価されたかを残す
- 誰かの過去経験・類似案件・失敗談をEvidenceとして残す
- 「実績がある」「実績はないが有望」等の不確実性を残す
- 未決事項・追加検証事項を残す
- 原文へ戻れるようにTimestampを残す
- AIが勝手に一般化・補完しない
- Transcriptにない技術的事実を追加しない

---

## 2. 入力

### 案件背景

以下を可能な範囲で事前に与える。

```yaml
case:
  case_id:
  product:
  product_category:
  usage:
  target_component:
  process:
  operation:
  equipment:
  workpiece:
  material:
  thickness:
  transportation:
  constraints:
```

### Teams会議情報

```yaml
meeting:
  meeting_id:
  date:
  participants:
  recording_url:
```

### Transcript

Teamsから取得した文字起こし全文を入力する。

---

## 3. AIへのプロンプト

以下をそのままベースプロンプトとして利用する。

---

あなたは、プレス金型・生産技術の設計検討会から、設計知識を構造化するアシスタントです。

目的は会議の要約ではありません。

会議中に議論された「設計Issue」を特定し、そのIssueについて、
複数人が出した設計案、各案への評価、過去経験、懸念、根拠、最終判断、未解決事項を、後から別の設計者が設計経緯を追えるレベルで整理してください。

## 基本ルール

1. 会議をIssue単位に分割してください。
2. 一つのIssueに複数のProposalがある場合、必ず別々に残してください。
3. 採用されなかったProposalも削除しないでください。
4. Proposalに対する肯定・否定・条件付き評価をEvaluationとして分けてください。
5. 過去案件、実験、解析、失敗経験、個人の経験則、定量値が出た場合はEvidenceとして記録してください。
6. 「実績あり」「実績なし」「推測」「要確認」など、発言の確実性を保持してください。
7. 決定していないものを決定済みにしないでください。
8. 発言から判断理由が明確でない場合は null としてください。
9. Transcriptに存在しない知識を補完してはいけません。
10. 重要な発言には開始時刻・終了時刻を付けてください。
11. 発言者が特定できる場合は記録してください。
12. 同じProposalに対し意見が対立している場合、その対立を統合せず両方残してください。
13. 設計意図を理解するうえで重要な数値、公差、材料、寸法、設備条件は省略しないでください。
14. 最後に「このIssueからKnowledge候補として一般化できそうな内容」を出してください。ただし確定Knowledgeにはせず、候補として記述してください。

---

## 出力フォーマット

```markdown
# 設計検討会 構造化記録

## Meeting
- Meeting ID:
- Case ID:
- Date:
- Participants:
- Recording:
- Summary:

---

## Issue 1

### Issue
- Issue ID:
- Title:
- Category:
- Status: open / decided / pending / experiment_required
- Background:
- Design objective:
- Constraints:
- Related components:
- Related process:
- Related assets:

### Proposal 1
- Proposal ID:
- Name:
- Description:
- Proposed by:
- Timestamp:

#### Evaluation
- Positive points:
- Negative points:
- Conditions for success:
- Risks:
- Uncertainties:
- Feasibility:
- Novelty:
  - established
  - proven_in_other_case
  - unproven_but_promising
  - experimental
  - unknown

#### Evidence
- Type:
  - past_case
  - experiment
  - simulation
  - production_result
  - drawing
  - cad
  - personal_experience
  - assumption
  - other
- Description:
- Source:
- Speaker:
- Timestamp:
- Reliability:
  - high
  - medium
  - low
  - unknown

### Proposal 2
...

### Discussion Dynamics

このIssueの議論がどのように進んだかを、時系列で簡潔に整理する。

例：
1. 当初A案が提示された
2. ○○氏から過去不具合の指摘
3. B案が代替案として提示
4. B案にも△△の懸念
5. 最終的にB案を採用し、△△を追加確認することになった

### Decision
- Selected proposal:
- Decision:
- Decision reason:
- Rejected proposals and reasons:
- Remaining risks:
- Open questions:
- Required follow-up:
- Decision timestamp:

### Knowledge Candidate
- Candidate title:
- Potential knowledge:
- Applies when:
- Does not necessarily apply when:
- Supporting evidence:
- Confidence:
- Notes:

---
```

---

## 4. 抽出時の重要ポイント

### 4.1 「案」を落とさない

悪い例：

```text
決定事項：
外形位置決めを採用。
```

良い例：

```text
Proposal A：
パイロット穴位置決め

良い点：
一般的で設計が単純。

懸念：
積層後の穴位置精度が保証できない。

Proposal B：
外形位置決め

良い点：
積層後ワーク自体を基準にできる。

懸念：
外形公差が位置決め精度を満足するか要確認。

Decision：
B案を採用。
```

### 4.2 経験則をEvidenceとして拾う

例：

> 「前の○○製品でもここが割れた」

は単なる雑談ではなく重要なEvidence候補。

以下を残す。

- 誰が言ったか
- どの案件か
- 何が起きたか
- 今回と何が同じ/違うか
- 根拠資料が存在するか
- 発言の信頼度

### 4.3 「攻めた案」を消さない

例：

- 性能上は魅力的
- 過去実績なし
- 開発日程上のリスクあり

といった案はKnowledge上重要である。

採用されなかったからといって削除しない。

### 4.4 不確実な発言を断定に変えない

例：

> 「過去にメッキ剥がれがあったらしい」

は、

```yaml
evidence_type: personal_experience
reliability: low
description: 過去にメッキ剥離問題があったとの発言
```

とし、

> メッキ端は3mm離すべき

という確定ルールに変換しない。

---

## 5. 人によるレビュー項目

AI生成後、設計者は最低限以下を確認する。

- Issueの切り方は妥当か
- Proposalの抜けがないか
- 採用/却下理由に誤りがないか
- 過去経験・実験・数値を落としていないか
- AIが勝手に補完していないか
- Timestampが適切か
- Knowledge Candidateが過度に一般化されていないか
