# CkipTagger
Also: [中文 README](https://github.com/ckiplab/ckiptagger/wiki/Chinese-README)

This open-source library implements neural CKIP-style Chinese NLP tools.
* (WS) word segmentation
* (POS) part-of-speech tagging
* (NER) named entity recognition

Related demo sites
- [CkipTagger](http://ckip.iis.sinica.edu.tw/service/ckiptagger)
- [CKIPWS (classic)](http://ckipsvr.iis.sinica.edu.tw)
- [CKIP CoreNLP](http://ckip.iis.sinica.edu.tw/service/corenlp)

Features
- +1.4%/+4.0%/+2.2% performance vs. classic CKIPWS(/POS/NER) on ASBC4.0/OntoNotes5.0
- Do not auto delete/change/add characters
- Support indefinitely long sentences
- Support user-defined recommended-word list and must-word list

## Installation

tl;dr.
```
pip install -U ckiptagger[tf,gdown]
```

CkipTagger is a Python library hosted on PyPI. Requirements:
- python>=3.6
- tensorflow / tensorflow-gpu (one of them)
- gdown (optional, for downloading model files from google drive)

(Minimum installation) If you have set up tensorflow, and would like to download model files by your self.
```
pip install -U ckiptagger
```

(Complete installation) If you have just set up a clean virtual environment, and want everything, including GPU support.
```
pip install -U ckiptagger[tfgpu,gdown]
```

## Usage

Complete demo script: demo.py. The following sections assume:
```python
from ckiptagger import data_utils, construct_dictionary, WS, POS, NER
```

### 1. Download model files

The model files are available on several mirror sites.
- [iis-ckip](http://ckip.iis.sinica.edu.tw/data/ckiptagger/data.zip)
- [gdrive-ckip](https://drive.google.com/drive/folders/105IKCb88evUyLKlLondvDBoh7Dy_I1tm)
- [gdrive-jacobvsdanniel](https://drive.google.com/drive/folders/15BDjL2IaX3eYdFVzT422VwCb743Hrbi3)

You can download and extract to the desired path by one of the included API.
```python
# Downloads to ./data.zip (2GB) and extracts to ./data/
# data_utils.download_data_url("./") # iis-ckip
data_utils.download_data_gdown("./") # gdrive-ckip
```
- ./data/model_ner/pos_list.txt -> POS tag list, see [Wiki](https://github.com/ckiplab/ckiptagger/wiki/POS-Tags) / [Technical Report no. 93-05](http://ckip.iis.sinica.edu.tw/CKIP/tr/9305_2013%20revision.pdf)
- ./data/model_ner/label_list.txt -> Entity type list, see [Wiki](https://github.com/ckiplab/ckiptagger/wiki/Entity-Types) / [OntoNotes Release 5.0](https://catalog.ldc.upenn.edu/docs/LDC2013T19/OntoNotes-Release-5.0.pdf)
- ./data/embedding_* -> character/word embeddings

### 2. Load model
```python
ws = WS("./data")
pos = POS("./data")
ner = NER("./data")
```

### 3. (Optional) Create dictionary

You can supply words for WS speicial consideration, including their relative weights.
```python
word_to_weight = {
    "土地公": 1,
    "土地婆": 1,
    "公有": 2,
    "": 1,
    "來亂的": "啦",
    "緯來體育台": 1,
}
dictionary = construct_dictionary(word_to_weight)
print(dictionary)
```
```
[(2, {'公有': 2.0}), (3, {'土地公': 1.0, '土地婆': 1.0}), (5, {'緯來體育台': 1.0})]
```

### 4. Run the WS-POS-NER pipeline
```python
sentence_list = [
    "傅達仁今將執行安樂死，卻突然爆出自己20年前遭緯來體育台封殺，他不懂自己哪裡得罪到電視台。",
    "美國參議院針對今天總統布什所提名的勞工部長趙小蘭展開認可聽證會，預料她將會很順利通過參議院支持，成為該國有史以來第一位的華裔女性內閣成員。",
    "",
    "土地公有政策?？還是土地婆有政策。.",
    "… 你確定嗎… 不要再騙了……",
    "最多容納59,000個人,或5.9萬人,再多就不行了.這是環評的結論.",
    "科長說:1,坪數對人數為1:3。2,可以再增加。",
]

word_sentence_list = ws(
    sentence_list,
    # sentence_segmentation=True, # To consider delimiters
    # segment_delimiter_set = {",", "。", ":", "?", "!", ";"}), # This is the defualt set of delimiters
    # recommend_dictionary = dictionary1, # words in this dictionary are encouraged
    # coerce_dictionary = dictionary2, # words in this dictionary are forced
)

pos_sentence_list = pos(word_sentence_list)

entity_sentence_list = ner(word_sentence_list, pos_sentence_list)
```

### 5. (Optional) Release memory
```python
del ws
del pos
del ner
```

### 6. Show Results
```python
def print_word_pos_sentence(word_sentence, pos_sentence):
    assert len(word_sentence) == len(pos_sentence)
    for word, pos in zip(word_sentence, pos_sentence):
        print(f"{word}({pos})", end="\u3000")
    print()
    return
    
for i, sentence in enumerate(sentence_list):
    print()
    print(f"'{sentence}'")
    print_word_pos_sentence(word_sentence_list[i],  pos_sentence_list[i])
    for entity in sorted(entity_sentence_list[i]):
        print(entity)
```
```

'傅達仁今將執行安樂死，卻突然爆出自己20年前遭緯來體育台封殺，他不懂自己哪裡得罪到電視台。'
傅達仁(Nb)　今(Nd)　將(D)　執行(VC)　安樂死(Na)　，(COMMACATEGORY)　卻(D)　突然(D)　爆出(VJ)　自己(Nh)　20(Neu)　年(Nf)　前(Ng)　遭(P)　緯來(Nb)　體育台(Na)　封殺(VC)　，(COMMACATEGORY)　他(Nh)　不(D)　懂(VK)　自己(Nh)　哪裡(Ncd)　得罪到(VJ)　電視台(Nc)　。(PERIODCATEGORY)　
(0, 3, 'PERSON', '傅達仁')
(18, 22, 'DATE', '20年前')
(23, 28, 'ORG', '緯來體育台')

'美國參議院針對今天總統布什所提名的勞工部長趙小蘭展開認可聽證會，預料她將會很順利通過參議院支持，成為該國有史以來第一位的華裔女性內閣成員。'
美國(Nc)　參議院(Nc)　針對(P)　今天(Nd)　總統(Na)　布什(Nb)　所(D)　提名(VC)　的(DE)　勞工部長(Na)　趙小蘭(Nb)　展開(VC)　認可(VC)　聽證會(Na)　，(COMMACATEGORY)　預料(VE)　她(Nh)　將(D)　會(D)　很(Dfa)　順利(VH)　通過(VC)　參議院(Nc)　支持(VC)　，(COMMACATEGORY)　成為(VG)　該(Nes)　國(Nc)　有史以來(D)　第一(Neu)　位(Nf)　的(DE)　華裔(Na)　女性(Na)　內閣(Na)　成員(Na)　。(PERIODCATEGORY)　
(0, 2, 'GPE', '美國')
(2, 5, 'ORG', '參議院')
(7, 9, 'DATE', '今天')
(11, 13, 'PERSON', '布什')
(17, 21, 'ORG', '勞工部長')
(21, 24, 'PERSON', '趙小蘭')
(42, 45, 'ORG', '參議院')
(56, 58, 'ORDINAL', '第一')
(60, 62, 'NORP', '華裔')

''


'土地公有政策?？還是土地婆有政策。.'
土地公(Nb)　有(V_2)　政策(Na)　?(QUESTIONCATEGORY)　？(QUESTIONCATEGORY)　還是(Caa)　土地(Na)　婆(Na)　有(V_2)　政策(Na)　。(PERIODCATEGORY)　.(PERIODCATEGORY)　
(0, 3, 'PERSON', '土地公')

'… 你確定嗎… 不要再騙了……'
…(ETCCATEGORY)　 (WHITESPACE)　你(Nh)　確定(VK)　嗎(T)　…(ETCCATEGORY)　 (WHITESPACE)　不要(D)　再(D)　騙(VC)　了(Di)　…(ETCCATEGORY)　…(ETCCATEGORY)　

'最多容納59,000個人,或5.9萬人,再多就不行了.這是環評的結論.'
最多(VH)　容納(VJ)　59,000(Neu)　個(Nf)　人(Na)　,(COMMACATEGORY)　或(Caa)　5.9萬(Neu)　人(Na)　,(COMMACATEGORY)　再(D)　多(D)　就(D)　不行(VH)　了(T)　.(PERIODCATEGORY)　這(Nep)　是(SHI)　環評(Na)　的(DE)　結論(Na)　.(PERIODCATEGORY)　
(4, 10, 'CARDINAL', '59,000')
(14, 18, 'CARDINAL', '5.9萬')

'科長說:1,坪數對人數為1:3。2,可以再增加。'
科長(Na)　說(VE)　:1,(Neu)　坪數(Na)　對(P)　人數(Na)　為(VG)　1:3(Neu)　。(PERIODCATEGORY)　2(Neu)　,(COMMACATEGORY)　可以(D)　再(D)　增加(VHC)　。(PERIODCATEGORY)　
(4, 6, 'CARDINAL', '1,')
(12, 13, 'CARDINAL', '1')
(14, 15, 'CARDINAL', '3')
(16, 17, 'CARDINAL', '2')

```

## Model Details

Please see:

Peng-Hsuan Li, Tsu-Jui Fu, and Wei-Yun Ma. 2019. [Remedying BiLSTM-CNN Deficiency in Modeling Cross-Context for NER](https://arxiv.org/abs/1908.11046). arXiv preprint arXiv:1908.11046.

## LICENSE

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />
Copyright 2019 CKIP under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/).

