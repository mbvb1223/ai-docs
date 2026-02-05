# Static Pages

## Overview

Static pages in Tempest allow you to generate pre-compiled HTML files that can be served directly by your web server without needing to boot the entire framework. This approach improves performance for pages without dynamic components.

To mark a controller action as a static page, use the `#[StaticPage]` attribute:

```php
use Tempest\Router\Get;
use Tempest\Router\StaticPage;
use Tempest\View\View;
use function Tempest\View\view;

final readonly class FrontPageController
{
    #[StaticPage]
    #[Get('/')]
    public function frontpage(): View
    {
        return view('./front-page');
    }
}
```

Generate and manage static pages using these commands:

```bash
./tempest static:generate
./tempest static:clean
```

The `static:clean` command removes all HTML files and empty directories from your `/public` directory.

## Data Providers

Most pages require dynamic data. Static pages support data providers that generate multiple pages from a single controller action.

Implement the `DataProvider` interface to specify which parameter combinations should be generated:

```php
use Tempest\Router\DataProvider;

final readonly class ChapterDataProvider implements DataProvider
{
    public function __construct(
        private ChapterRepository $chapters
    ) {}

    public function provide(): Generator
    {
        foreach ($this->chapters->all() as $chapter) {
            yield [
                'category' => $chapter->category,
                'slug' => $chapter->slug,
            ];
        }
    }
}
```

Attach the provider to your controller action:

```php
#[StaticPage(ChapterDataProvider::class)]
#[Get('/{category}/{slug}')]
public function show(string $category, string $slug, ChapterRepository $chapters): View
{
    // Controller logic
}
```

Container-injected dependencies are automatically resolved; only yield the route parameters.

## Crawling for Dead Links

Optionally scan for broken links during generation:

```bash
./tempest static:generate --crawl
```

Check external links as well:

```bash
./tempest static:generate --crawl --external
```

By default, only internal links are checked.

## Production

Static pages are generated as `index.html` files in the `/public` directory. Web servers automatically serve these without additional configuration.

Include `./tempest static:generate` in your deployment pipeline to ensure static pages are regenerated with each deployment.
