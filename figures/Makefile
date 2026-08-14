TEX_FILES := $(wildcard *.tex)
PDF_FILES := $(TEX_FILES:.tex=.pdf)
PNG_FILES := $(TEX_FILES:.tex=.png)

PDFLATEX       ?= pdflatex
PDFLATEX_FLAGS := -interaction=nonstopmode -halt-on-error
PDFTOCAIRO     ?= pdftocairo
PNG_DPI        := 200

.PHONY: all png clean

all: $(PDF_FILES)

png: $(PNG_FILES)

%.pdf: %.tex
	$(PDFLATEX) $(PDFLATEX_FLAGS) $<
	@rm -f $*.aux $*.log $*.out $*.fls $*.fdb_latexmk $*.synctex.gz

%.png: %.pdf
	$(PDFTOCAIRO) -png -r $(PNG_DPI) -singlefile $< $*

clean:
	rm -f $(PDF_FILES) $(PNG_FILES)
