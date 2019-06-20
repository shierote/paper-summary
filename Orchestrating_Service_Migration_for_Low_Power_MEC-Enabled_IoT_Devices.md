# Orchestrating Service Migration for Low Power MEC-Enabled IoT Devices
低電力MEC対応IoTデバイスのためのサービス移行をオーケストリングする。
https://arxiv.org/pdf/1905.12959.pdf

## Abstract
- Multi-Access Edge Computing (MEC): 5Gモバイルネットワークの鍵となる技術。
  - 分散クラウドコンピューティングの容量やモバイルネットワークの先端のアプリケーションやサービスの環境を促進する
  - cf. [マルチアクセスエッジコンピューティングとは](http://doge-man.com/2019/03/08/%E3%82%B5%E3%82%AF%E3%83%83%E3%81%A8%E8%A7%A3%E8%AA%AC%EF%BC%9A%E3%83%9E%E3%83%AB%E3%83%81%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%82%A8%E3%83%83%E3%82%B8%E3%82%B3%E3%83%B3%E3%83%94%E3%83%A5%E3%83%BC/)

- タイトルのデモを行う(以下のものを用いる)
  - [Kubernetes](https://kubernetes.io/ja/)
  - LTEのデバイスとMECサーバ
  - リアルタイムコミュニケーションAPIを提供する[WebRTC](https://webrtc.org/)

## 1. Introduction
- 5G MECサービスはモバイルネットワークオペレータのエージェントやオーケストレーターとして考えることができ、リソースのスケジューリングやアプリケーションの移行を可能にする。
- しかし開発者はMECのホストプロセスをMECホスト間でやりとりできるように設計しなければならない。
  - 待ち時間を担保するため
  - 無線のリソースを最適化するため

- オーケストレータにより管理されているコンテナで動作するマイクロサービスでも同様の課題が見られる。
  - cf. [Microservices](https://martinfowler.com/articles/microservices.html): 複数の小さなサービスをAPIにより連携させる(それぞれはコンテナ上で動作)
  - 大規模なエンジニアチームによりビジネスコンポーネントのスケーリングの課題とアジャイル開発に対処するため文書化されている。
  - コンテナを簡素化することにより(APIの)インターフェースとデプロイのされ方についても制限され合意がなされている
  - これによりデータセンターのハードウェアとオーケストレータのソフトウェア両方がコンテナの実行時間の仕様に準拠しているかぎり変更可能であるため保守性と信頼性が向上する

- 5GではSDNとNFV(ネットワーク機能仮想化)へ移行
  - cf. [SDN](https://www.infraexpert.com/study/sdn01.html)
  - CapEx DSPハードウェア→コンピューティングベースのセルラーネットワークアーキテクチャへ進化
   - MNOにおける費用を下げる

- MECパラダイムはARやVR、自動運転、MNO間のリソース市場を実現するものとして引用される

## 2. Background
- IoTデバイスの特徴
  - 低消費電力
  - 計算能力が限られている
  - 様々なワイヤレスセンサーやカメラなどが
- 石油やガスなどの分野で応用可能
  - これらの分野では遠隔である程度自律的に動作する必要がある
  - 計算上の負荷を軽減して、周囲のエネルギー収集技術を用いて装置を増強しIoTのシナリオを最適化しようとしている
- IoTシステムの効果的な運用のボトルネック
  - 待ち時間（latency）
  - モビリティ管理
  - ロケーション認識
  - データセンター内のサーバと直接する従来のアプローチは効率が悪い
- MECがIoTデバイスクラスターとクラウドサーバ間の中間層として機能する必要性あり

### A. 低電力MEC対応IoTデバイス
- MECの価値
  - 低遅延
  - 高帯域幅
  - 計算能力の向上
- MECがIoTの可能性を向上させる
  - エッジコンピューティングがエンドユーザーのデバイスリソースを最適化する
- 今日のようなオーケストレーションはIoTノード間の設定、信頼性、拡張性、セキュリティなど様々な対処が必要であるため将来のIoTアプリケーションのニーズに答えることができなさそう

### B. LTEハンドオーバー
- cf. [ローミングとハンドオーバーはどう違うか](https://tech.nikkeibp.co.jp/it/atcl/column/14/228621/092600005/)
- 以下の式で説明されるシンプルなイベントにより引き起こされる
  - Mn > Ms + HOmargine (1)
  - Mn: 隣接セルに対するdBm単位の基準信号受信力(RSRP): 電波の強さ(RSRQは基地局の混雑具合)
  - Ms: サービングセルのRSRP
  - HOmargine: MnとMsのマージン
  - 各セルはHOmargineの値を持っており、(1)に基づいてハンドオーバーの決定をする
  - 基地局(BS)が管理
  - X2インターフェースはBS間で情報を共有する、無線リソース管理プロセスにも影響
  - S1インターフェースはLTEのモビリティ管理エンティティ(MME)サーバを接続、負荷を分散

### C. コンテナベースのマイクロサービスアーキテクチャ  
- 近年クラウドサービスはモノリシック（一枚板）アーキテクチャからマイクロサービスアーキテクチャへと変化
  - マイクロサービスのメリット
    - メンテナンス性
    - 柔軟性
    - 拡張性
    - 効率性
    - シンプルにする
    - 独立して開発、テスト、デプロイ、スケール、運用、アップデートが可能
- MECはデータセンターとローカルデバイス間のクラウドコンピューティングのに新しい計算層をもたらす
  - 一部の機能をデータセンターからMECに移行することでネットワークリソースを節約し待ち時間を短くする
  - MECとマイクロサービスは相性が良い
- マイクロサービスはコンテナ技術が用いられる
  - Dockerなど
- オーケストレーションはコンテナなど仮想化されたコンポーネント間の相互作用を制御し、サービスの構成、管理、停止を処理するための技術
  - Docker Swarm、Kubernetes、Mesosなど

- 動的なサービス開発はコンテナおよび動的なオーケストレーションを通して実行されるマイクロサービスアーキテクチャを利用して実現
  - ユーザーの端末がRANを横切って移動するときに端末の近くのMECホストの最適な場所にサービスのインスタンスを展開する

### D. Related work

- [17]では将来の5Gや膨大なネットワークのデータ量と低遅延に対処するためにエッジコンピューティングとフォグコンピューティングを共同でオーケストレーションしようとしている

- [18]ではモバイルエッジクラウドにおけるアプリケーションオーケストレーションについて議論されている(アプリケーションのライフサイクルの管理効果などに焦点)

- [19]、[20]では無線アクセスネットワークの端にある、低い移動度のマシンタイプコミュニケーション(MTC)デバイスのグループのための拡張性を保証するための異なるメカニズムを議論している

- [21]ではコアネットワークとクラウドデータセンターに負担をかける可能性がある不要な通信を減らすためにデータトリミングを通してIoT通信をフィルタリングするためのスマートゲートウェイソリューションを提案

- [22]ではedgeIoTと呼ばれるエッジIoTアーキテクチャが伝統的なIoTアーキテクチャでスケーラビリティの問題に対処するためにモバイルエッジでデータストリームを扱うために提案

## 3. System Framework
- リアクティブMECサービスの移行をビデオストリーミングアプリケーションにより示す。
- Kubernetesにより管理されているMECサーバが着信ビデオにグレスケールフィルターを適用
- ユーザーの端末がカラーのビデオストリームをキャプチャし、MECであと処理されたものとオリジナルのビデオを並べて表示するWebアプリケーションを実装
- MECのE2Eの待ち時間を記録
- ソースコード: https://github.com/toldjuuso/handover2019okwuibe
- 手順
  - 図2では二つのBSと、二つのMECサーバがある
  - 最初にBS1に割り当てられたMEC AppをMEC1で実行
  - 次に2番目のBSに移動
  - EPC(Evolved Pocket Core)で認識されたS1ハンドオーバー要求が発生
  - EPCによりKubernetesサービス移行をMEC1からMEC2へリアクティブに移行
  - 移行によりグレースケールのビデオストリームの停止と切断
  - Kubernetesで移行して新しいホストでMECアプリケーションを起動
  - DNSユーザー端末で使用される1番目のMECサービスのアドレスから新しいホストを指すように更新
  - 接続が再確立され、グレースケールビデオストリームが続行される
- 今回のアプリケーションは以前の研究である[23]のものを拡張しており、[23]ではDevOps機能を備えたオープンソースのRANが提示されている。
- EPCはユーザー端末のトラフィック処理のためネットワーク内のKubernetesとユーザー端末間でリクエストをルーティングできる
- X2ではなくS1を利用しているのはMMEも実行しているEPCがネットワークトラフィックを利用してKubernetesを介したMECアプリケーション移行を統合できるため

### A. クライアントアプリケーションの設計
- Fig1の右側
- WebブラウザかはgeUserMediaAPIを利用してライブカメラフィードにアクセス
- WebRTCを利用してカメラフィードのユーザーデータグラムプロトコル(UDP)ストリームをMECに送信

### B. サーバアプリケーションの設計
- Fig1の左側
- (MEC)サーバ側WebRTC上のRTPストリームを受け取る
- サーバはIVFコンテナにデータをカプセル化してFFmpegでパイプ処理
- FFmpegはビデオにグレースフィルタを適用
- localhostソケットのUDP上でストリームとして送信
- MECアプリケーションはローカルUDPソケットを読み取りWebRTCを介してクライアントに送り返すためのビデオサンプルを作成する


### C. リアクティブサービスの移行
- MECオーケストレータとしてKubernetesを利用
- EPCとしてOSSであるNextEPCを使用
- EPCがハンドオーバー要求をログに記録すると、スクリプトがKubernetesスケジューラへ自動サービス移行をトリガーする
- KubernetesスケジューラはMEC1のPodを停止
- MEC2に新しいPod(Dockerを管理するための最小単位)を置き換える
- 移行を完了するとビデオpost-processサービスのIPエンドポイントを更新してサービスを再び利用可能にする

## 4. Evaluation And Results

### A. End-to-End Latency
- 遅延はユーザー端末の前でストップウィッチにより測定
- ライブカメラフィードと処理後のフィードの両方を示す画面のスクリーンショットを撮り表示されているストップウォッチの時間をの差分から取得
- 結果はFig3
  - 最小遅延: 0.09秒
  - 最大遅延: 1.52秒
  - 平均: 0.55秒

### B. エネルギー消費

- エネルギー消費量は電力計により測定
  - 何もしていない状態で2.1Wh
  - ビデオのエンコード中に 6.5Wh
- デバイス上で完結させるより約1/3消費電力を削減できる

### C. アプリケーション移行待ち時間

- 待ち時間はKubernetesのログから測定
  - アプリケーション移行の遅延の95%は6.805秒
  - 最小遅延: 2.730秒
  - 最大遅延: 7.480秒
  - 平均: 4.450秒

### 無線信号強度

- ハンドオーバーを定義するためにBSがEPCからEnd Markerを受信した時のタイムスタンプを記録
- Fig4とFig5に記録
-

## 5. Conclusion

- この研究ではMECアプリケーションの移行をS1無線ハンドオーバーに統合
  - オーケストレータシステムを統合することの実用上の問題、ユーザーの端末がMECのリソースに依存することへ対処した
- Kubernetesを利用すると平均4.450秒のコンテナ移行時間（遅い）が出ており、将来のMEC対応携帯端末ネットワークではアプリケーション移行に対して新しいアプローチが必要になる可能性がある
  - エッジコンピューティングやフォグコンピューティングに依存するアプリケーションは深刻なダウンタイムが出る恐れがありサービスの品質を下げる可能性がある
  - これに対処するため機械学習を利用して事前にコンテナを移動させ移行時間に対処できるように研究を進めていく