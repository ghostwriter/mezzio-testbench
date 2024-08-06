# Mezzio TestBench

[![Automation](https://github.com/ghostwriter/mezzio-testbench/actions/workflows/automation.yml/badge.svg)](https://github.com/ghostwriter/mezzio-testbench/actions/workflows/automation.yml)
[![Supported PHP Version](https://badgen.net/packagist/php/ghostwriter/mezzio-testbench?color=8892bf)](https://www.php.net/supported-versions)
[![GitHub Sponsors](https://img.shields.io/github/sponsors/ghostwriter?label=Sponsor+@ghostwriter/mezzio-testbench&logo=GitHub+Sponsors)](https://github.com/sponsors/ghostwriter)
[![Code Coverage](https://codecov.io/gh/ghostwriter/mezzio-testbench/branch/main/graph/badge.svg)](https://codecov.io/gh/ghostwriter/mezzio-testbench)
[![Type Coverage](https://shepherd.dev/github/ghostwriter/mezzio-testbench/coverage.svg)](https://shepherd.dev/github/ghostwriter/mezzio-testbench)
[![Psalm Level](https://shepherd.dev/github/ghostwriter/mezzio-testbench/level.svg)](https://psalm.dev/docs/running_psalm/error_levels)
[![Latest Version on Packagist](https://badgen.net/packagist/v/ghostwriter/mezzio-testbench)](https://packagist.org/packages/ghostwriter/mezzio-testbench)
[![Downloads](https://badgen.net/packagist/dt/ghostwriter/mezzio-testbench?color=blue)](https://packagist.org/packages/ghostwriter/mezzio-testbench)

Provides a testing environment for Mezzio applications using PHPUnit.

> [!WARNING]
>
> This project is not finished yet, work in progress.

## Installation

You can install the package via composer:

``` bash
composer require ghostwriter/mezzio-testbench
```

### Star ⭐️ this repo if you find it useful

You can also star (🌟) this repo to find it easier later.

## Usage

Extend `\Ghostwriter\MezzioTestBench\AbstractTestCase` instead of `PHPUnit\Framework\TestCase`

```php
use Ghostwriter\MezzioTestBench\AbstractTestCase;
use Mezzio\Application;
use Mezzio\MiddlewareFactory;
use Mezzio\Template\TemplateRendererInterface;
use Psr\Container\ContainerInterface;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;

final class HomePageTest extends AbstractTestCase
{
    /**
     * Setup the test environment.
     */
    protected function setUp(): void
    {
        $this->beforeApplicationCreated(function () {
            // Run before application created.
        });

        $this->afterApplicationCreated(function (
            RouterInterface $router,
            // inject other dependencies here
        ) {
            // Run after application created.
            $router->get('/', function (RequestInterface $request): ResponseInterface {
                return new Response(200, [], '#BlackLivesMatter');
            }, 'home');

            $router->post('/posts/{postId}/comments', function (RequestInterface $request): ResponseInterface {
                static $comments = [];

                $postId = $request->getAttribute('postId');
                $comments[] = $request->getParsedBody()['comment'];
                $commentId = count($comments);
                
                return new Response(201, ['Location' => sprintf('/posts/%d/comments/%d', $postId, $commentId)]);
            }, 'posts.comments.store');
        });

        $this->beforeApplicationDestroyed(function () {
            // Run before application destroyed.
        });
        
        $this->afterApplicationDestroyed(function () {
            // Run after application destroyed.
        });

        parent::setUp();
    }

    public function testHomePage(): void
    {
        $this->get('home');
        // or
        $this->get('/');

        $this->assertResponseStatusCode(200);
        $this->assertResponseBodyContains('#BlackLivesMatter');
    }
    
    public function testAddCommentToPost(): void
    {
        $uri = $this->router()->generateUri('posts.comments.store', ['postId' => 1]);
        $comment = ['comment' => '#BlackLivesMatter'];
        
        $this->post($uri, $comment);

        $this->assertResponseStatusCode(201);
        $this->assertResponseRedirectsTo('/posts/1/comments/1');
        // or
        $this->assertResponseRedirectsToRoute('posts.comments.show', ['postId' => 1, 'commentId' => 1]);
    }
}
```

### Credits

- [Nathanael Esayeas](https://github.com/ghostwriter)
- [All Contributors](https://github.com/ghostwriter/mezzio-testbench/contributors)

### Changelog

Please see [CHANGELOG.md](./CHANGELOG.md) for more information on what has changed recently.

### License

Please see [LICENSE](./LICENSE) for more information on the license that applies to this project.

### Security

Please see [SECURITY.md](./SECURITY.md) for more information on security disclosure process.
