# For details of the original implementation see this https://huggingface.co/openai/clip-vit-base-patch32/tree/main

# run this python script to generate tagset.txt and pos.dict.utf8 from vocab.json AND merges.txt
python export_vocab.py

# zip it the dictionary file
zip pos.dict.utf8.zip pos.dict.utf8

# make sure the tools are compiled and are in the path, see wiki for details

# build LDB as usual, you should get an output like below and no error messages (do "clean" target if encounter errors)
~/BlingFire/ldbsrc$ make -f Makefile.gnu lang=clip all 

fa_build_conf \
  --in=clip/ldb.conf.small \
  --out=clip/tmp/ldb.mmap.small.txt
fa_fsm2fsm_pack --type=mmap \
  --in=clip/tmp/ldb.mmap.small.txt \
  --out=clip/tmp/ldb.conf.small.dump \
  --auto-test
unzip -p clip/pos.dict.utf8.zip | \
fa_build_dict  --input-enc=DEC --type=mph --raw --tagset=clip/tagset.txt --float-nums \
  --out-fsm=clip/tmp/pos.dict.fsm.txt \
  --out-k2i=clip/tmp/pos.dict.k2i.txt \
  --out-i2info=clip/tmp/pos.dict.i2t.txt
fa_fsm2fsm_pack --alg=triv --type=mealy-dfa  --in=clip/tmp/pos.dict.fsm.txt --out=clip/tmp/pos.dict.fsm.small.dump --auto-test
fa_fsm2fsm_pack --alg=triv --type=arr --force-flat  --in=clip/tmp/pos.dict.k2i.txt --out=clip/tmp/pos.dict.k2i.small.dump --auto-test
fa_fsm2fsm_pack --alg=fixed --type=mmap  --in=clip/tmp/pos.dict.i2t.txt --out=clip/tmp/pos.dict.i2t.small.dump --auto-test
fa_merge_dumps --out=ldb/clip.bin clip/tmp/ldb.conf.small.dump clip/tmp/pos.dict.fsm.small.dump clip/tmp/pos.dict.k2i.small.dump clip/tmp/pos.dict.i2t.small.dump

# Testing

python3 code.py

tokenization without bpe:
['a', 'Ġphoto', 'Ġof', 'Ġa', 'Ġreally', ',', 'Ġfunctistaner', 'Ġbig', 'Ġcat', '.']
tokenization with bpe:
['a', 'Ġ', 'photo', 'Ġ', 'of', 'Ġ', 'a', 'Ġ', 'really', ',', 'Ġ', 'func', 'ti', 'stan', 'er', 'Ġ', 'big', 'Ġ', 'cat', '.']
Hugging face ids: 
[64, 220, 1153, 220, 684, 220, 64, 220, 43249, 11, 220, 8679, 555, 2203, 517, 220, 1915, 220, 1481, 13]
BlingFire ids: 
[ 320 1125  539  320 1414   11 1499   66  555 2203  517 1205 2368   13]
