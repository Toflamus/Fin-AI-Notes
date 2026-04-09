# Beamer 编译脚本
# 使用 xelatex 支持中文

MAIN = main
TEX  = xelatex
OPTS = -interaction=nonstopmode -halt-on-error

.PHONY: all clean watch

all: $(MAIN).pdf

$(MAIN).pdf: $(MAIN).tex
	$(TEX) $(OPTS) $(MAIN).tex
	$(TEX) $(OPTS) $(MAIN).tex  # 二次编译以生成 TOC

clean:
	rm -f $(MAIN).aux $(MAIN).log $(MAIN).nav $(MAIN).out \
	      $(MAIN).snm $(MAIN).toc $(MAIN).vrb $(MAIN).fdb_latexmk \
	      $(MAIN).fls $(MAIN).synctex.gz $(MAIN).xdv

watch:
	latexmk -pvc -xelatex $(MAIN).tex
