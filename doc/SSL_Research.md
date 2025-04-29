# 自己教師あり学習（Self-Supervised Learning; SSL）モデル開発の変遷とトレンド
## 1. なぜ SSL が求められるのか
ラベル付けコストを削減しながら、高性能な特徴表現を獲得できることから SSL は CV/NLP で急速に普及。近年は Vision Transformer (ViT) や Foundation Model の台頭とともに研究が加速している。最新のレビューでは、SSL の多様な目的関数・アーキテクチャが体系化されている。

## 2. 年代別マイルストーン (2014 → 2025)

| 項目 | SimCLR | SimSiam | DINO | DINO v2 |
|:-----|:-------|:--------|:-----|:--------|
| アーキテクチャ | CNN／ViT + 2 × MLP 双方向同重み | CNN／ViT + predictor 付き Siamese 片方向 stop-grad | ViT + Teacher / Student (EMA) | ViT (Base‒Giant) + EMA Teacher |
| 学習信号 | NT-Xent コントラスト損失：正例1 対 大量負例 | ネガティブ不要。片側勾配を止め崩壊回避 | Teacher 表現を Student が回帰（自己蒸留） | DINO 改良版＋重み正則化・大規模 clean データ |
| 特徴 | - 強力な拡張 + 大バッチ必須<br>- 実装シンプル | - バッチ小でも安定<br>- 実装最小限 | - マルチクロップで局所/大域特徴を同時学習<br>- セグメンテーション情報が自発的に出現 | - 1.4 億画像（LVD-142M）事前学習<br>- ゼロショット・線形評価とも SoTA |
| 長所 | • 理論的に直感的<br>• 多数派ベースライン | • 学習コスト低い<br>• モバイル向けに転用容易 | • ViT と相性抜群<br>• k-NN だけで ImageNet 80%↑ | • 汎用視覚基盤：医療・衛星など転移強い |
| 短所 | • バッチ ≥ 4K & メモリ重<br>• ネガティブ設計が複雑 | • まだ対比法より若干精度低<br>• Stop-grad 理解にコツ | • EMA 計算コスト↑<br>• 大規模 pretrain 前提 | • 事前学習コスト非常に高い<br>• 学習コード未完全公開 |
| 代表的発展形 | SimCLR v2, MoCo v3（メモリバンク） | BYOL, Guided Stop-Grad (GSG) arXiv | iBOT（MIM + DINO） | I-JEPA / V-JEPA（予測型 JEPA 系へ発展） |


## 3. トレンド
- 目的関数の進化

コントラスト (SimCLR) → ネガティブ不要 (BYOL/SimSiam) → 冗長性削減 (Barlow Twins) → マスク予測 (MAE) → 予測型 Joint Embedding (I-JEPA)。設計が 簡潔・安定 になりつつ、データスケールを伸ばせる方向へ。

- アーキテクチャの転換

CNN から ViT へ移行。ViT はパッチ表現とマスクタスクが自然に噛み合い、大規模事前学習で性能が線形にスケール。

- データと計算のスケール化

100 万枚規模 (SimCLR) → 1 億枚超 (DINOv2)。クラスタリング＋フィルタリングにより ノイズを抑えた超大規模画像コーパス が鍵。

- Foundation Model 化

学習済み重みを「汎用視覚エンジン」として再利用する流れ。DINOv2 は線形評価・下流ファインチューニングとも高性能で、医療や産業カメラ など異領域にもそのまま転移可能と報告。

- マルチモーダル・動画への拡張

CLIP 系の視覚-言語と並行し、V-JEPA など「時空間マスク＋予測」で動画理解へ拡大。将来は ロボティクス や 自律エージェント での自己学習が期待される。

## 4. 2025 年以降の注目ポイント
高解像度 & 3D 表現: NeRF/3D Gaussian との連携で空間理解を向上

合成データと自己拡張: 生成モデルでラベルフリー増強 → SSL との自己強化ループ

オンデバイス学習: 勾配レス蒸留・量子化が進み、組込みカメラでリアルタイム自己学習

産業応用: 製造検査・ロボット組立で「少量ラベル × Foundation SSL」が主流に

## 5. まとめ
自己教師あり学習は

1. 目的関数の単純化 と

2. Transformer＋大規模事前学習 の２軸で急速に進歩し、
2023 年以降は DINOv2 / I-JEPA 系 を中心に “視覚基盤モデル” へと進化した。
今後は マルチモーダル・動画・3D へ対象が広がり、産業現場でも少量データで高精度モデルを立ち上げるデフォルト戦略になるだろう。
https://arxiv.org/pdf/2408.17059
https://arxiv.org/html/2312.02366v4
https://arxiv.org/html/2411.09598v1

# SimCLR・SimSiam・DINO・DINOv2 ― 基本構造と相違点
| 項目 | SimCLR | SimSiam | DINO | DINO v2 |
|:-----|:-------|:--------|:-----|:--------|
| アーキテクチャ | CNN／ViT + 2 × MLP 双方向同重み | CNN／ViT + predictor 付き Siamese 片方向 stop-grad | ViT + Teacher / Student (EMA) | ViT (Base‒Giant) + EMA Teacher |
| 学習信号 | NT-Xent コントラスト損失：正例1 対 大量負例 | ネガティブ不要。片側勾配を止め崩壊回避 | Teacher 表現を Student が回帰（自己蒸留） | DINO 改良版＋重み正則化・大規模 clean データ |
| 特徴 | - 強力な拡張 + 大バッチ必須<br>- 実装シンプル | - バッチ小でも安定<br>- 実装最小限 | - マルチクロップで局所/大域特徴を同時学習<br>- セグメンテーション情報が自発的に出現 | - 1.4 億画像（LVD-142M）事前学習<br>- ゼロショット・線形評価とも SoTA |
| 長所 | • 理論的に直感的<br>• 多数派ベースライン | • 学習コスト低い<br>• モバイル向けに転用容易 | • ViT と相性抜群<br>• k-NN だけで ImageNet 80%↑ | • 汎用視覚基盤：医療・衛星など転移強い |
| 短所 | • バッチ ≥ 4K & メモリ重<br>• ネガティブ設計が複雑 | • まだ対比法より若干精度低<br>• Stop-grad 理解にコツ | • EMA 計算コスト↑<br>• 大規模 pretrain 前提 | • 事前学習コスト非常に高い<br>• 学習コード未完全公開 |
| 代表的発展形 | SimCLR v2, MoCo v3（メモリバンク） | BYOL, Guided Stop-Grad (GSG) arXiv | iBOT（MIM + DINO） | I-JEPA / V-JEPA（予測型 JEPA 系へ発展） |



## point
[SimCLR](https://arxiv.org/abs/2002.05709?utm_source=chatgpt.com) = “正と負を遠ざける” 伝統的コントラスト学習。

[SimSiam](https://arxiv.org/abs/2011.10566) = “負例ゼロでもいける” 極小構成。

[DINO](https://arxiv.org/abs/2104.14294?utm_source=chatgpt.com) = “自分で自分を教師にする” 蒸留アプローチ。

[DINO v2](https://ai.facebook.com/blog/dino-v2-computer-vision-self-supervised-learning/?utm_source=chatgpt.com) = “巨大データ＋強化学習則” で Foundation Model 化。



# 動画からの作業分類 ― 最適 SSL モデル
## 2.1 現状と課題
現行手法：YOLO + Pose による姿勢推定 → ルールで作業ラベル付け

問題点：
<br>
① ラベル整備が専門家依存／高コスト
<br>
② 作業バリエーション増でルール爆発
<br>
③ 現場利用者が ML 知識なしでも扱える簡便性が必要

## 2.2 推奨パイプライン
| ステージ | 推奨モデル | 目的 |
|:---------|:-----------|:-----|
| ① 自己教師ありプレトレ | [VideoMAE（Masked Autoencoder for Video）](https://link.springer.com/article/10.1007/s00138-024-01638-9) または [V-JEPA](https://ai.facebook.com/blog/v-jepa-yann-lecun-ai-model-video-joint-embedding-predictive-architecture/?utm_source=chatgpt.com) | 90%以上のチューブマスクで時空間再構成 → 時系列特徴を学習 |
| ② 軽量タスクヘッド | 時間 Transformer (TimesFormer Tiny) ＋ 線形分類器 | 少量ラベルで各組立ステップを微調整 |
| ③ 作業区間推定 | 不連続点検出 + 簡易 HMM／MSTCN | 特徴勾配を用い作業開始・終了を自動分割 |
| ④ 現場 UI | Edge-TSP / ONNX Runtime | 推論 30fps以上をノートPCで実現し、ダッシュボードに作業時間を可視化 |

### 理由
- VideoMAE は “大量のラベル無し動画” からデータ効率良く表現を学習し、産業ラインにドメイン適応した[実績あり](https://link.springer.com/article/10.1007/s00138-024-01638-9)

- ネガティブ不要で学習が安定、固定カメラ映像とも相性◎。

- 部品が小さく写る場合でもマスク再構成タスクが細部特徴を強制的に拾う。

- 微調整段では 十数本のラベル付き動画 で高精度 (>90 %) 報告例がある。

## 2.3 長所と期待効果
- 省ラベル：現場作業員が 1 〜 2 時間アノテーションするだけで足りる

- 自動工程時間計測：推論後に連続同ラベル区間の長さを積算

- メンテ容易：作業追加時は新動画を放り込み再プレトレ or 継続学習

## 2.4 残課題
- 計算資源：VideoMAE プレトレは GPU メモリ 32 GB クラス推奨

- ドメインドリフト：新治工具・照明変化 ⇒ 定期的に無ラベル動画で継続学習

- アクションの粒度定義：作業工程の階層（工程→作業→モーション）設計が先行タスク

- プライバシー／安全：顔・名札等の自動マスキング処理が必要



#  6大 SSL モデルの特徴・長所・短所・主な用途

| モデル | 方式カテゴリ | 主要アイデア (学習信号) | 要件・コスト | 長所 | 短所 | 代表的ターゲット |
|:------|:-------------|:------------------------|:-------------|:-----|:-----|:----------------|
| SimCLR | ⚔️ コントラスト | NT-Xent：正例1 + 多数負例。強拡張 & 大バッチで特徴を引き離す | バッチ≳4096、メモリ大 | 実装シンプル／理論直感的 | ネガティブ設計と大計算資源が必須 | 静止画表現基盤（ImageNet など） [arXiv](https://arxiv.org/) |
| SimSiam | 🪞 非コントラスト | Stop-gradient + 予測 MLP。負例もモメンタムも不要 | 小バッチでOK／学習高速 | 超軽量・崩壊しにくい | 最高精度は SimCLR 未満 | モバイル／エッジでの事前学習 [arXiv](https://arxiv.org/) |
| DINO | 🐣 自己蒸留 | EMA Teacher ↔️ Student が表現を一致させる | ViT + マルチクロップ、EMA 計算増 | ViT と相性抜群／セグメ情報が自発的に浮上 | 大規模 pretrain 前提 | 汎用視覚表現（検出・分割） [arXiv](https://arxiv.org/) |
| DINO v2 | 🐣📏 拡張自己蒸留 | DINO改良 + LVD-142M (1.4億画像) でスケール | 巨大データ + 多GPU | Foundation Model化：医療・衛星等へゼロショット転移◎ | 学習コスト極大／weights 440M〜2B | 業務横断の視覚基盤 [arXiv](https://arxiv.org/) |
| VideoMAE | 🎭 マスク再構成 | 動画チューブを90%隠しピクセル再構成 | ViT + 16 frame/clip, 計算効率高 | 小データでも高性能／データ効率◎ | デコーダ分の計算あり | アクション認識・動画理解 [arXiv](https://arxiv.org/) |
| V-JEPA | 🔮 予測 Joint Embedding | コンテキストブロックからターゲット表現を予測（非生成） | ViT、小バッチ可 | マスク要らず／抽象的表現を直接学習 | 公開 weights まだ限定／実装例少 | 将来型：動画世界モデル・RL [arXiv](https://arxiv.org/) |

## 分類の見方

- コントラスト系（SimCLR）→ ネガティブ不要系（SimSiam）→ 蒸留系（DINO）→ 大規模蒸留（DINO v2）と “負例削減・スケール化” が軸。

- 一方、マスク予測 (VideoMAE) と 予測 JEPA (V-JEPA) は 動画 に拡大し「未来／欠損部分を補う」ことで時空間理解を得る。
