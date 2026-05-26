# This Makefile shows how to "compile" tex source into
# a PDF. It has also targets for cleaning temporary
# files (clean) that are generated while processing
# tex file. Also, there's a target for cleaning everything
# which includes PDF as well (clean-all).

all: clean-all paper.pdf clean

paper.pdf: paper.tex references.bib
	pdflatex paper.tex
	bibtex paper
	pdflatex paper.tex
	pdflatex paper.tex

clean:
	rm -f missfont.log paper.log paper.aux paper.dvi paper.aux paper.bbl paper.blg

clean-all: clean
	rm -f paper.pdf
