# Для локальной отладки поднимает на localhost:4000

`docker run -p 4000:4000 -v $(pwd):/site bretfisher/jekyll-serve`


// cmd `jekyll serve` have problem with ruby on macos