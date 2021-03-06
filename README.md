# NLG-System
A natural language generation system to generate technical docs

This code uses [OpenAi gpt-2](https://github.com/openai/gpt-2)
and also [finetuning by nshepperd](https://github.com/nshepperd/gpt-2/tree/finetuning)
## Preprocessing
<details>
<summary>Things you can or should do before training.</summary>

#### Download Model with:
Available are Modells "117M" and "354M" (not tested) adjust output_dir in script!
> python 1Preprocessing\download_model.py 117M

Be careful we added a Language-identifier to the Hyperparams in h_params.json. Please add "h_params" = "en"

#### Create encoder.json and vocab.bpe
Use subword-nmt by Rico Sennrich to create new Byte Pair Encoding for your Language.
1. Place a .txt File you want to extract embeddings from in data/embedding
2. start process with
    > subword-nmt learn-joint-bpe-and-vocab --input data/embedding/yourfile.txt --output data/embedding/vocab.txt --write-vocabulary data/embedding/encoder.txt --separator Ġ --symbols 50257 -v
3. Reformat Output so it fits gpt-2
    > python 1Preprocessing/format_embeddings.py
4. Move encoder.json and vocab.bpe to your base language-model in directory models

#### Convert Trainingdata PDFs to txt
1. Place PDFs in training/PDF
2. Clean PDFs with own rules (regex, str.replace) in pdf_to_txt.py
3. Use pdf_to_txt.py to parse PDFs to txt-File (with Apache Tika)
    > python 1Preprocessing/pdf_to_txt.py

#### Create .npz
If you don't want to encode your Trainingdata on every run, you can save it encoded with numpy savez and load from that file.
> python 1Preprocessing\pre_encode.py .\data\training\PDF .\data\training\trainingsdaten.npz --model_name ISW_Model
</details>
 
## Training (based on nshepperd)
<details>
<summary>Steps to train your own model</summary>

1. We recommend to parse your file into single .txt (see Preprocessing)
2. Pre-Encode to npz (recommended see Preprocessing)
3. download model to retrain and rename it
4. Create Embeddings (encoder.json and vocab.bpe) for your language (optional, not recommended because of Problems)
5. replace encoder and vocab files
6. start retraining with:
    > python 2Training/train.py --dataset ./data/training/trainingsdaten.npz --model_name ISW_Model --sample_every 100 --sample_length 200 --run_name iswTrain1
7. wait
8. if you are satisfied with samples (data/training/samples) and loss: stop (ctrl+c)
9. get newest checkpoint from data/training/checkpoint/runX
10. replace the following files in your model with the new ones
        
        * checkpoint
        * model.ckpt.data-00000-of-00001
        * model.ckpt.index
        * model.ckpt.meta

11. Adjust hparams.n_lang to your language
12. your model is ready to use. If you want to see some stats on tensorboard use:
    > tensorboard  --logdir=data/training/iswTrain1/checkpoint

</details>

## Backend (based on OpenAi GPT-2)
<details>
<summary>See what the Backend does</summary>

[Uses Huggingface pytorch-Transformer](https://huggingface.co/transformers/index.html)

Communicates with Frontend via REST-API on Flask server.  
Receives supporting Words which the Text should contain and Settings from User-Input in Frontend.

Implements 4 different Strategies to build Text from given supporting Words:


>1a. Beam-Search  
>1b. Beam-Search with Scope  
>2.Search until fit  
>3.Cut-off and insert  
>4.BERT-GPT2 Hybrid

For Details see paper.
</details>

## Frontend
<details>
<summary>How to use the Word-Add-In</summary>

Generated with Yeoman-Generator for Office-Add-ins

For changes edit React App in 4Frontend/src/taskpane/components

To sideload your Add-In in Word use the following command inside of directory 4Frontend
> npm start

and

>npm stop

Open  Start > Show TDTG > Enter your Inputs and Settings > click generate

Wenn Änderungen, die Sie am Manifest vorgenommen haben, z. B. Dateinamen von Symbolen für Schaltflächen im Menüband anscheinend nicht wirksam werden, löschen Sie den Office-Cache auf Ihrem Computer.
Löschen des Inhalts des Ordners %LOCALAPPDATA%\Microsoft\Office\16.0\Wef\

Be careful with spaces in your Pathname, they are followed by errors with webpack loading the CA-Certificate and block sideloading your Add-In
</details>

