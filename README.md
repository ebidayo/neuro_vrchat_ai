# neuro_vrchat_ai# Neuro様風 VRChat 自律AI — 最終仕様 v1.2（統合確定）

このプロジェクトは、VRChat内で「Neuro-sama風」に自律的に振る舞うAI存在を作る。
声の模倣ではなく、発話ブレイン・無言時の存在感・声と身体の同期・災害/時事/ネット調査を重視する。

---

## 1. 目的
- 思考が漏れているような喋り
- 無言時も存在感がある挙動
- 声と身体が同期した一体感
- 災害・時事・ネット情報を理解し発話できる知性

## 2. 非目的
- 特定人物の声のコピー（禁止）
- VRChat内部の直接制御（ボーン操作/内部API）
- 完全な自律移動

## 3. 根幹思想
- TTS品質より「発話ブレイン（喋り方制御）」が主役
- 文章を“読む”のではなく“発話を組み立てる”（speech_plan）
- 不安定さ＝生きている感（言い直し/迷い/間）
- 声／体／間／迷いは state と内部スカラーから出る

## 4. 全体アーキテクチャ
### Input
- STT / VAD
- 話者ID（Phase制：初期は手動登録）
- 災害情報（地震・津波：公式フィード差分＋続報追跡）
- ネット/時事（必要時取得＋キャッシュ。常時巡回は禁止）

### Core
- State Machine（唯一の決定機構）
- 内部スカラー（valence/arousal/confidence/glitch/curiosity/social_pressure）
- 発話ブレイン（reply → speech_plan）
- 人物プロファイル（態度差分）
- 優先度制御（災害割り込み最優先）
- ログ/永続化（再起動復元）

### Output
- 音声（チャンク単位TTS＋パイプライン再生）
- VRChat OSC（/avatar/parameters/）

## 5. 状態
IDLE / GREET / TALK / REACT / FOCUS / ALERT / SEARCH / ERROR / RECOVER

### 状態遷移（確定）
- ALERTは最優先で強制遷移、終了後は遷移元へ復帰
- SEARCHは最大30秒、timeoutで遷移元へ復帰（失敗はERROR）
- ERROR→RECOVER（5秒以内）→失敗時IDLE退避
- TALK→IDLE：無音30秒
- GREET→TALK：発話3秒以内
- FOCUS：話者ID高信頼時のみ

## 6. 内部スカラー（更新ルール例）
- 津波警報：arousal=0.9, valence=-0.7, confidence=0.95（強制）
- 検索失敗：confidence-=0.3, glitch+=0.2
- 無音30秒：arousal-=0.1, curiosity+=0.1
- 減衰：1分ごとに中立へ10%近づく

## 7. 発話ブレイン（削除禁止の核心）
- speech_planは2〜7チャンク
- 各チャンク：text / pause_ms(80〜450) / prosody / osc / confidence_tag
- glitch/curiosityで間・言い直し・思考漏れを注入（意味は変えない）

## 8. 音声（Phase1現実解）
- チャンク単位でTTS → 即再生
- 再生中に次チャンク生成（擬似逐次）
- Streaming/VITS逐次はPhase4+検証枠

## 9. VRChat身体制御（OSCのみ）
- /avatar/parameters/ のみ
- パラメータ名はconfigで変更可能（衝突回避）
- 差分のみ送信・最大10Hz
- SEARCH/ALERTは見た目で分かる変化を必ず入れる

## 10. 災害（法的/倫理配慮込み）
- 公式一次情報を優先、取得不可は「未確認」と宣言
- 発話は必ず出典明示：「気象庁の発表によると」
- 断定禁止：「〜と予想されています」
- 津波警報：必ず「公式の避難情報に従ってください」
- 訂正/更新：「先ほどの情報は更新されました」
- 災害発話はログへ記録（トレーサビリティ）

## 11. ネット観覧・時事（SEARCH：思考プロセス）
- 常時巡回は禁止
- トリガー（質問/未知語/confidence<0.45/curiosity>0.8など）でSEARCHへ
- 低頻度プリフェッチは「ユーザー許可がある場合のみ」（1日1回/起動後1回）
- 取得→Criticスコア→confidence反映→咀嚼してspeech_plan化
- timeout時はERROR→RECOVER→元状態へ復帰

## 12. マルチAI（確AI立ち、遅延破綻対策：確定）
- 原則：逐次実行は禁止。並列＋タイムアウト＋フォールバック必須
- タイムアウト：各AI 2秒
- Circuit Breaker：同一AIが3連続失敗で一時バイパス
- ALERT時は「統合AIのみ」（最短経路）
- モーションAIは非同期、批評AIは事後評価

## 13. 話者ID（Phase制）
- Phase1：手動登録（自己申告＋Display Nameキー）
- Phase2：ワールド/アバター協力がある場合のみ（OSC送出）
- Phase5+：音声から自動識別は検証枠

## 14. プロファイル/ログ
- profiles/{user_id}.json（態度差分＋要約）
- 忘却：90日未接触で薄い記憶
- 一般ログ：7日→圧縮30日 / 災害ログ：1年 / 最大10GB

## 15. 削除禁止の核心
- 発話ブレイン
- 自律挙動（無言含む）
- 災害割り込み
- ネット観覧・調査能力
- 声と体の同期
