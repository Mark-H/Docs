---
title: "Scheduled Newsletter"
translation: "extending-modx/creating-components/scheduled-newsletter"
---

The last lesson will be the shortest. It was unexpectedly discovered that we have already done everything, and all that remains is to add a script for sending letters on a schedule.

Just in case, I remind the algorithm of the component:

* Create a newsletter and specify its properties. Be sure to specify a template or snippet.
* We add users (or they are added independently, through the website)
* On the page of the queue of letters we add new ones by selecting the newsletter. Depending on what is indicated in it, the text of the letter is generated by a snippet or template.
* We send letters. You can manually from the admin, or a script on a schedule.

Everything except the script, we have already done.

## Scheduled emails

Given that everything we need is already written in the `sxQueue` class, you only need to connect the component model, select the records and call the `send()` method.

``` php
<?php
// This is for calling from the development directory.
if (file_exists(dirname(dirname(dirname(dirname(dirname(__FILE__))))) . '/config.core.php')) {
    require_once dirname(dirname(dirname(dirname(dirname(__FILE__))))) . '/config.core.php';
}
else {
    require_once dirname(dirname(dirname(dirname(dirname(dirname(__FILE__)))))) . '/config.core.php';
}

// Get the config and call MODX
require_once MODX_CORE_PATH . 'config/' . MODX_CONFIG_KEY . '.inc.php';
require_once MODX_CONNECTORS_PATH . 'index.php';

// Add a model
$modx->addPackage('sendex', MODX_CORE_PATH . 'components/sendex/model/');

// Choose 100 letters
$q = $modx->newQuery('sxQueue');
$q->limit($modx->getOption('sendex_queue_limit', null, 100, true));

// We send
$queue = $modx->getCollection('sxQueue');
/** @var sxQueue $email */
foreach ($queue as $email) {
    $email->send();
}
```

After sending the letter is deleted, so the script can be run at least every 2 minutes - it will not create a load.

I added this file to `/core/components/sendex/cron/send.php`, so that it cannot be called from the browser. Why give the opportunity to not understand who send your letters?

You can add it to cron or manually (see your hosting documentation). You usually need to log in to the server via SSH and type **crontab -e** and add in the editor:

``` php
*/2  *  * * *   php /var/www/site/www/core/components/sendex/cron/send.php
```

The server will pull the file every 2 minutes, and all mailings will fly away instantly. At the MODXCloud laboratory tariff, I could not log in to SSH, probably disconnected.

[Вот коммит с файлом send.php](https://github.com/bezumkin/Sendex/commit/52b15960eeddb2968bc1585f9e6f630c8b59438f).

In principle, that’s all the last lesson, but I’ll add some more on the topic of adding letters to the queue through the API.

## Adding emails via API

I don’t know how it was done in other solutions, but I don’t imagine how to automate the addition of letters to the newsletter.

After all, there can be a huge number of situations and conditions. Sending new tickets from the questions section, or news with template 5 from parent 13 - this is not all drawn in the interface.

Therefore, the automatic addition of letters to the list we leave on the conscience of the site developer. We have already prepared everything for him:

* Create a snippet or template that will form the body of the letter
* Create a newsletter and specify a snippet or template
* Get the `sxNewsletter` object and call the `addQueues` method

That is, everything is very simple.

Just in case, here is the code:

``` php
$modx->addPackage('sendex', MODX_CORE_PATH . 'components/sendex/model/');

/** @var sxNewsletter $newsletter */
if ($newsletter = $modx->getObject('sxNewsletter', array('name' => 'Newsletter'))) {
    $newsletter->addQueues();
}
```

You can call this code when publishing new pages, according to a schedule or something else. In your snippet or template, you can select any documents and under any conditions. In general, you can fill the mailing with any kind of information.

I think this is quite a promising design that will be easy and pleasant to use. Moreover, we wrote everything to manage mailings manually.

## Conclusion

At the same time our course ends.

Ask questions, we will discuss incomprehensibility. In the near future, I will try to review the code, maybe a little refine the logic and put it in the repository.

All changes can be [monitored on GitHub](https://github.com/bezumkin/Sendex/commits/master).
