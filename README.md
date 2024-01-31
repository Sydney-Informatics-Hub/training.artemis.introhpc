# Introduction to Artemis High Performance Computer

## File location

- Store `index.qmd`, `setup.qmd` files in the root directory 
- Store notebooks (.ipynb), R files (as .qmd or .rmd), and markdown lessons (.qmd or .md) in the `notebooks` folder
- Store figures in a `figs` folder 

## How to use this repo 

1. Edit `index.qmd` to change the main landing page. This is basically a markdown file.
2. Edit or create `setup.qmd` to change the Setup instruction pages. Same - basically a md file.
3. Edit `_quarto.yml` to change the dropdown menu options.
4. Add additional `*.md` files to the root dir to have them converted to html files (and add them to `_quarto.yml` to make them navigable), if you'd like.
5. Edit this Readme in your fork to reflect the content of your workshop.

The project will be built and rendered automatically (via github actions) at a URL with this format:

[https://sydney-informatics-hub.github.io/training.artemis.introhpc/](https://sydney-informatics-hub.github.io/training.artemis.introhpc/)

To get this working you must manually make a new branch called `gh-pages` and then adjust on GitHub Settings > Pages > Build and Deployment, to `Source: Deploy from a branch` and `gh-pages: /(root)`. You may also have to do a `quarto render` once offline before pushing to main.


 

