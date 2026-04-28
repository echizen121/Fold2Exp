# Fold2Exp

## 概要
Alpha　Fold2やRoseTTAFoldはタンパク質構造予測を大きく前進させたが、依然として計算資源にかかる問題は大きなボトルネックとして残留している。
本プロジェクトでは、とくにAlphaFold　Datebase（以下AFDBと略）に公開されている予測構造とRCSB　PDBに公開されている実験構造およびメタデータを比較し、そのズレを定量化することにより、構造予測AIがどのような配列、構造条件で実験構造から乖離しやすいのかを解析する。

## 構成
本プロジェクトでは、RCSB　PDB由来の実験構造データをＥ、AFDB由来の予測構造データをFとする。
また、情報取得用の関数群として、それぞれ
RCSB　PDBから実験構造Eを取得するePicker,AFDBから予測構造Fを取得するfPickerがある。

## 処理の流れ
ePickerにはRCSB Search API、RCSB Data API、RCSB File Download Servicesを用い、fPickerには
ePicker は、まずRCSB Search APIにより条件に合うprotein polymer entityを検索し、pdb_id と entity_id を取得する。次に、RCSB Data APIを用いてentry metadataおよびpolymer entity metadataを取得する。さらに、RCSB File Download ServicesからmmCIF形式の実験構造ファイルを取得し、これらの情報を E として整理する。

E には、主にPDB ID、entity ID、chain ID、実験法、分解能、配列、配列長、構造ファイルの保存パスなどを記録する。

次に、SIFTSを参照し、E に含まれるPDB chainがどのUniProt accessionに対応するかを取得する。ここでは、pdb_chain_uniprot.tsv.gz によりPDB chainとUniProt accessionの対応を取得し、uniprot_segments_observed.tsv.gz により実験構造中で実際に観測された残基範囲を取得する。
fPicker は、SIFTSにより得られたUniProt accessionを用いて、AFDBから対応する予測構造を取得する。AFDB metadataから構造ファイルURLなどを取得し、予測構造ファイルをローカルに保存する。取得結果は F として整理する。

F には、主にUniProt accession、AFDB ID、予測構造ファイルの保存パス、metadataの保存パス、取得状態などを記録する。

その後、E と F について、uniprot_segments_observed により報告された観測済み範囲を比較対象とし、対応する残基・座標を用いてズレ指標を計算する。対象とする指標には、グローバルRMSD、局所RMSD、ドメイン角度、Ramachandran、側鎖差、radius_of_gyration差、end-to-end distance差、GDT相当などが含まれる。
最後に、各E-Fペアについて、これら複数のズレ指標をまとめ、ズレ指標ベクトルとして扱う。これにより、実験構造と予測構造の差を単一の値ではなく、複数の観点から記録する。
