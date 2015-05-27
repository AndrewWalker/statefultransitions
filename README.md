# Stateful transitions blog

My blog is online at http://andrewwalker.github.io/statefultransitions

## Dependencies

- [hugo](http://gohugo.io/) v0.13

## Usage

Running the hugo server

```shell
hugo server --buildDrafts --watch
```

Adding new content

```shell
hugo new post/something.md
```

Publishing to gh-pages branch

```shell
git subtree push --prefix=public git@github.com:AndrewWalker/statefultransitions.git gh-pages
```
