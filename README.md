# DupliCap
Automatic conversion of the internal representation of a capacitor V3.14

## About this repository
This repository is for distributing the Spice models and related files introduced in the YouTube video.

## このリポジトリについて
このリポジトリは、YouTube動画で紹介した Spice モデルや関連ファイルを配布するためのものです。

## 関連動画
【伝スパ】LTSpiceとPythonrV3.14で自動生成ムラタ・ニチコン・ニッケミコンデンサの内部表現モデル アップデート#3  
https://youtu.be/HeDfZtg57_0

## ダウンロード
配布ファイルは Releases から取得してください。

## 内容
1. このプログラムの目的

dupliCap は、メーカーが提供するコンデンサ詳細モデルを LTspice 上でシミュレーションし、その特性を LTspice の内部表現モデルへ移し替えるための支援プログラムです。

メーカー詳細モデルは精密ですが、回路全体のシミュレーションでは重くなりやすく、解析時間が長くなることがあります。
本プログラムでは、メーカー詳細モデルの周波数特性から等価な内部表現用パラメータを求め、より軽量で高速に扱える内部表現モデルを作成します。

本案件の主な目的は次の通りです。

- メーカー詳細モデルを読み出す
- 型番から容量、耐圧、シリーズなどの情報を自動抽出する
- 詳細モデルを LTspice でシミュレーションする
- 内部表現モデル側の ESR、ESL などを合わせ込む
- 結果を一覧ファイルとして出力し、後段で利用できるようにする


# dupliCap User Guide

## 1. Purpose of This Program

**dupliCap** is a support tool for simulating manufacturer-provided detailed capacitor models in LTspice and transferring their characteristics into LTspice's internal capacitor representation model.

Manufacturer detailed models are precise, but they can become heavy in full-circuit simulations, resulting in longer analysis times.  
This program derives equivalent parameters for LTspice's internal representation from the frequency characteristics of the manufacturer’s detailed model, allowing the creation of a lighter and faster internal representation model.

The main objectives of this project are as follows:

- Read manufacturer-provided detailed models
- Automatically extract information such as capacitance, voltage rating, and series from the part number
- Simulate the detailed model in LTspice
- Fit internal representation parameters such as ESR and ESL
- Output the results as a list file for downstream use
