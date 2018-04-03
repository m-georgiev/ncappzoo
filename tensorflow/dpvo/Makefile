
IMGSIZE=299
MODEL_CKPT_TAR=inceptionv3_meta.tar.gz
MODEL_CKPT=InceptionV3_noBN
OUTPUT_NODE_NAME='predictions/Softmax'

YELLOW='\033[1;33m'
NOCOLOR='\033[0m'

.PHONY: all
all: profile

check_model:
ifndef TF_MODELS_PATH
	$(error TF_MODELS_PATH is not defined. Run `export TF_MODELS_PATH=path/to/your/tensorflow/models/repo`)
endif

check_tf:
ifndef TF_SRC_PATH
	$(error TF_SRC_PATH is not defined. Run `export TF_SRC_PATH=path/to/your/tensorflow/source/repo`)
endif

.PHONY: checkpoint
checkpoint:
	@echo $(YELLOW)"\nDownloading checkpoint files..."$(NOCOLOR)
	(mkdir -p model)
	test -f ./model/$(MODEL_CKPT_TAR) || (cd model; wget 'https://www.dropbox.com/s/z8laup5hxq7s6ov/$(MODEL_CKPT_TAR)' -O $(MODEL_CKPT_TAR);)
	(cd model; tar -xvf $(MODEL_CKPT_TAR);)

.PHONY: compile
compile: checkpoint
	@echo $(YELLOW)"\nCompiling model to Movidius graph..."$(NOCOLOR)
	(cd model; mvNCCompile -s 12 $(MODEL_CKPT).meta -is $(IMGSIZE) $(IMGSIZE) -on=$(OUTPUT_NODE_NAME);)
	@echo $(YELLOW)"Movidius graph generated! You can run inferences using ncappzoo/apps/image-classifier project.\n"$(NOCOLOR)

.PHONY: profile
profile: checkpoint
	@echo $(YELLOW)"\nProfiling the model..."$(NOCOLOR)
	(cd model; mvNCProfile -s 12 $(MODEL_CKPT).meta -in=input -on=$(OUTPUT_NODE_NAME);)
	@echo $(YELLOW)"Movidius graph generated! You can run inferences using ncappzoo/apps/image-classifier project.\n"$(NOCOLOR)

.PHONY: check
check: checkpoint
	@echo $(YELLOW)"\nComparing results with standard Tensorflow..."$(NOCOLOR)
	(cd model; mvNCCheck -s 12 $(MODEL_CKPT).meta -in=input -on=$(OUTPUT_NODE_NAME);)
	@echo $(YELLOW)"Movidius graph generated! You can run inferences using ncappzoo/apps/image-classifier project.\n"$(NOCOLOR)

.PHONY: run
run: compile
	@echo $(YELLOW)"\nRunning inferences using image-classifier project..."$(NOCOLOR)
	(python3 ../../apps/image-classifier/image-classifier.py --graph ./model/graph --dim $(IMGSIZE) $(IMGSIZE) --mean 127.5 --scale 0.00789 --colormode "RGB" --labels ./model/labels.txt --image ../../data/images/pic_053.jpg)

.PHONY: help
help:
	@echo $(YELLOW)"\nPossible make targets: "$(NOCOLOR);
	@echo "  make help - Shows this message.";
	@echo "  make - Builds all dependencies, but does not run this program.";
	@echo "  make checkpoint - Downloads pre-trained checkpoint files.;"
	@echo "  make compile - Convert the frozen model into Movidius graph file.";
	@echo "  make check - Compare inference results with that of TensorFlow running on CPU/GPU.";
	@echo "  make profile - Run the model on NCS and extract complexity, bandwidth and execution time for each layer.";

.PHONY: clean
clean:
	@echo $(YELLOW)"\nMaking clean..."$(NOCOLOR);
	rm -rf model
