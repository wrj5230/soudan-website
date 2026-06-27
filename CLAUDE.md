# CLAUDE.md — soudan-website プロジェクト

## プロジェクト概要
オーダーメイド仏壇のコンフィギュレーター（相談ウェブサイト）。
ゲームの車カスタマイズUIのようなイメージで、仏壇のパーツをクリックして色を選べる。

## 現在の状態（2026-06-23 作業終了時点）

### フェーズ1：2D SVGコンフィギュレーター ほぼ完成
`src/pages/configurator.astro` に実装済み。

**実装済み機能：**
- 仏壇SVGの4パーツ（外枠・台 / 扉（表） / 扉（裏） / 背板）をクリックで選択
- パーツ選択時に選択箇所以外を暗転させる選択エフェクト（SVGマスク使用）
- 選択箇所に白いボーダーハイライト
- 扉（裏）選択時：扉が172度まで開いて450msでカラースワップ
- 扉グループ内にハイライトを置いて回転と一緒に動くようにしてある
- 「全体を確認」ボタン（preview-headerの中に配置、暗転エフェクトOFF）
- カラーパレット（漆黒/朱/深緋/群青/常磐/紫苑/飴色/藍鉄/金/白木/溜塗/カスタム）
- カラーサマリー（下部のドット表示）
- シャッフルボタン
- ステップ2：部屋配置（Stable Diffusion連携予定）

**レイアウト：**
- グリッド `360px 1fr`（左パネル固定、プレビューパネルが残り）
- altar-wrap max-width: 620px
- altar-inner padding: 0 90px（扉開閉時の見切れ対策）
- SVGとaltar-wrapはoverflow: visible

**選択エフェクトの仕組み：**
- `mask-outer-dim`：外枠以外を暗くする（外枠選択時）
- `mask-inner-dim`：内側以外を暗くする（扉・背板選択時）
- `#dim-outer` / `#dim-inner`：暗転オーバーレイrect（マスクで制御）
- `#hl-outer` / `#hl-door-front` / `#hl-back`：白い枠線
- `#hl-door-left-back` / `#hl-door-right-back`：扉グループ内に配置（回転と同期）

**扉アニメーションの仕組み（重要）：**
- CSS: `perspective(900px) rotateY(-172deg)` / `rotateY(172deg)`
- 0.9秒アニメーションの450ms時点（扉が真横で見えない瞬間）に fill色をスワップ
- `door-left-front` / `door-right-front` のfillを表裏で切り替える
- SVGはpreserve-3dが使えないためこの方式を採用

**試したが却下した方法：**
- 扉に装飾（金フレーム・彫りパネル）→ 172度で残像・横棒が出るため削除
- opacity:0で扉を消して inner rect を表示 → 扉が消えて見える問題
- 扉を90度で止めてフェードイン → 「悪くなった」
- カードフリップ方式 → 「元のが良い」と戻した
- 選択ハイライトをゴールド色 → 選択中の色がわからなくなる
- 白いモヤ（gaussian blur） → 白すぎて色が見えなくなる

---

## 次のフェーズ：Three.js 3D化（たたき台完成 2026-06-15）

**理由：** 2D SVGでは艶感・扉の表裏表現・奥行き感に限界がある

**やりたいこと：**
- 仏壇の3Dモデルをくるくる回せる（OrbitControls）
- 漆塗りの艶感を表現（MeshPhysicalMaterial + clearcoat）
- パーツごとに色変更（現行機能を3Dで再現）

**計画：**
1. Blenderで仏壇3Dモデルを作成
   - Blender Pythonスクリプトで基本形を自動生成
   - パーツ（外枠/扉L/扉R/背板/金具）をメッシュ単位で分けて命名する
   - GLTF/GLB形式でエクスポート
2. Three.jsをAstroに組み込む（npm install three）
3. GLTFLoaderでモデル読み込み
4. OrbitControlsで回転
5. MeshPhysicalMaterialで漆塗り質感
6. HDRIで環境反射
7. メッシュ名でパーツ検索 → マテリアル色変更

**現在の状況：** モバイルUI作業完了・保留中（2026-06-27）
- altar.glb：`public/altar.glb` に配置済み
- Three.js実装済み：OrbitControls / 扉開閉アニメーション / 表裏別マテリアル / カラーパレット連動
- デバッグ用 `console.log('mesh:', obj.name)` が残っているので本番前に削除

**モバイルUI実装済み（2026-06-27時点）：**
- ボトムシート（スワイプ開閉・ハンドル付き）
- 4ステップ構成：Step1カスタマイズ → Step2プレビュー → Step3概算 → Step4部屋配置
- Step2でカメラズームイン演出（z=3.6 → z=3.2）
- 扉・台はタップで開閉（レイキャスト）、扉開放時に台も連動
- カラードットをプレビュー上部に固定表示
- Step3の概算合計を palette-scroll 外に固定
- Step3・4でスワイプ閉じ無効（ハンドルタップのみ閉じ）
- カメラ位置：モバイル初期(0, 0.5, 3.6) / Step2(0, 0.4, 3.2) / Step3以降は元に戻る

**保留中のiPhone対応（未解決）：**
- アドレスバー分のレイアウト調整（dvh / bottom offset など試したが改善見られず）
- ngrok経由での確認のため実機検証が難しい状況

**次のステップ：**
- ライティング調整（より漆感を出す）
- Blenderモデルを本格化（装飾・金具）
- HDRIで環境反射を追加
- デプロイ（Vercel）・ドメイン取得

**技法の状態（2026-06-23）：**
- 蒔絵 ✅ / ラップ塗り ✅ / 螺鈿 🔒非表示（職人調整待ち）
- 技法UIは扉（表・裏）と背板のみ表示、外枠・垂れ幕では非表示

**蒔絵テクスチャ（2026-06-23 更新）：**
- 金粉粒子ベース（微粒子3900個 + 大粒子190個）＋Perlinノイズ密度ムラ
- repeat.set(5,5) でタイリング、emissiveMapとして使用
- params: fineSize=0.9 / bigSize=2.5 / glow=0.5 / glowAlpha=0.27 / variation=0.59 / varScale=7.9

**ラップ塗りテクスチャ（2026-06-23 更新）：**
- Perlinノイズ（FBM 5オクターブ）+ sineのマーブル筋で実装済み
- `createRaptoTexture` 関数に `_perm/_fade/_grad2/_noise2/_fbm` ヘルパーを追加
- パラメーター: nx*5スケール / n*2.5歪み / sine絶対値 / べき乗3でシャープ化

**次回最初に実行するスクリプト（Blender Scripting → New → 貼り付け → Run）:**
```python
import bpy

bpy.ops.object.select_all(action='SELECT')
bpy.ops.object.delete()

W=0.60; H=0.90; D=0.38; t=0.022; bH=0.14; cH=0.048
iH=H-bH-cH; iW=W-2*t; door_W=iW/2

def add_box(name,cx,cy,cz,sx,sy,sz):
    bpy.ops.mesh.primitive_cube_add(size=1,location=(cx,cy,cz))
    obj=bpy.context.active_object; obj.name=name
    obj.dimensions=(sx,sy,sz); bpy.ops.object.transform_apply(scale=True)
    return obj

def to_col(obj,col):
    for c in list(obj.users_collection): c.objects.unlink(obj)
    col.objects.link(obj)

def set_origin(obj,x,y,z):
    bpy.ops.object.select_all(action='DESELECT'); obj.select_set(True)
    bpy.context.view_layer.objects.active=obj
    bpy.context.scene.cursor.location=(x,y,z)
    bpy.ops.object.origin_set(type='ORIGIN_CURSOR')

col=bpy.data.collections.new("仏壇")
bpy.context.scene.collection.children.link(col)

body_H=H-bH; body_z=bH+body_H/2; door_y=D/2-t/2
parts=[
    add_box("_base",0,0,bH/2,W,D,bH),
    add_box("_lwall",-(W/2-t/2),0,body_z,t,D,body_H),
    add_box("_rwall",(W/2-t/2),0,body_z,t,D,body_H),
    add_box("_top",0,0,H-cH/2,W-2*t,D,cH),
    add_box("_back",0,-(D/2-t/2),body_z,W,t,body_H),
]
for p in parts: to_col(p,col)
bpy.ops.object.select_all(action='DESELECT')
for p in parts: p.select_set(True)
bpy.context.view_layer.objects.active=parts[0]
bpy.ops.object.join()
bpy.context.active_object.name="outer_frame"

bp=add_box("back_panel",0,-(D/2-t*2),bH+iH/2,iW,t,iH); to_col(bp,col)
dL=add_box("door_L",-(iW/2-door_W/2),door_y,bH+iH/2,door_W,t,iH); to_col(dL,col)
set_origin(dL,-iW/2,door_y,bH+iH/2)
dR=add_box("door_R",(iW/2-door_W/2),door_y,bH+iH/2,door_W,t,iH); to_col(dR,col)
set_origin(dR,iW/2,door_y,bH+iH/2)
bpy.context.scene.cursor.location=(0,0,0)
print("完了: outer_frame / back_panel / door_L / door_R")
```

---

## 技術スタック
- フレームワーク：Astro v6
- 開発サーバー：http://localhost:4323/configurator
- Stable Diffusion API：http://127.0.0.1:7860（--apiフラグ未追加）

## ファイル構成（主要）
```
src/pages/configurator.astro   ← メインファイル（全実装）
src/styles/global.css          ← グローバルCSS（--color-gold: #c9a84c など）
public/                        ← 静的ファイル
```
