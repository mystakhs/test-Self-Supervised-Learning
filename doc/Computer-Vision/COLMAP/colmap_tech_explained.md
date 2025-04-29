# 📸 COLMAPによるSfM技術解説

## 概要

**COLMAP**は、画像群からカメラポーズ（位置・姿勢）や3D点群を自動的に復元する**Structure-from-Motion (SfM)**エンジンです。研究・産業の両方で広く用いられています。

- 公式リポジトリ: [colmap/colmap](https://github.com/colmap/colmap)
- 関連論文: [Schönberger and Frahm, "Structure-from-Motion Revisited," CVPR 2016](https://arxiv.org/abs/1606.01221)

---

## 🔧 COLMAPの構成技術

### 1. キーポイント検出と記述

- 特徴点（例: SIFT）を用いて画像中の特徴的な領域を検出し、128次元のディスクリプタで記述
- 代表論文: [Lowe, "Distinctive Image Features from Scale-Invariant Keypoints," IJCV 2004](https://www.cs.ubc.ca/~lowe/papers/ijcv04.pdf)

### 2. 特徴点マッチング

- 異なる画像間の対応点をマッチング
- Ratio test（Loweの比率テスト）で誤対応を除去

### 3. 幾何学的検証

- **RANSAC**を用いて不正なマッチングを除去
- Fundamental / Essential行列を推定

### 4. Structure-from-Motion（SfM）処理

以下をインクリメンタルに構築：

1. **相対カメラポーズの推定**（Essential matrixの分解）
2. **三角測量（Triangulation）**による3D点群の生成
3. **PnP**による新しい画像の追加
4. **バンドル調整（Bundle Adjustment）**による最適化

- バンドル調整についての詳細: [Triggs et al., "Bundle Adjustment — A Modern Synthesis," 2000](https://www.robots.ox.ac.uk/~vgg/presentations/bundle2000/bundle-ijcv.pdf)

### 5. Multi-View Stereo (MVS)

- PatchMatchアルゴリズムを用いて、密な点群を生成

---

## 🔌 pycolmapについて

COLMAPをPythonから操作可能にするラッパーです。Kaggleなどの自動化パイプラインに有用。

- pycolmap GitHub: [colmap/pycolmap](https://github.com/colmap/pycolmap)

---

## 🧠 Kaggle Image Matching Challenge 2025での使い方

1. ALIKEDでローカル特徴量抽出（[aliked](https://github.com/ducha-aiki/ALIKED)）
2. LightGlueでマッチング（[LightGlue](https://github.com/cvg/LightGlue)）
3. `h5_to_db`ツールでCOLMAP DBへ変換
4. `pycolmap`を使って再構築（回転R・並進Tの取得）

---

## 📘 関連技術

- SIFT: https://docs.opencv.org/4.x/d5/d3c/classcv_1_1SIFT.html
- RANSAC: https://en.wikipedia.org/wiki/Random_sample_consensus
- Essential Matrix分解: Hartley & Zisserman, *Multiple View Geometry*
- Bundle Adjustment: https://github.com/colmap/colmap/blob/dev/scripts/python/bundle_adjuster.py

---

## 📝 参考資料・論文

- [COLMAP 公式論文 (CVPR 2016)](https://arxiv.org/abs/1606.01221)
- [COLMAP GitHub](https://github.com/colmap/colmap)
- [pycolmap GitHub](https://github.com/colmap/pycolmap)
- [LightGlue GitHub](https://github.com/cvg/LightGlue)
- [ALIKED GitHub](https://github.com/ducha-aiki/ALIKED)