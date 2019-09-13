#!/usr/bin/env php
<?php

checkVendors();

function checkVendors()
{
    if (!file_exists(__DIR__.'/vendor'))
    {
        try {
            echo "Trying to install all components...\n";
            $cmd = "composer install";
            exec(sprintf('%s  2>&1', $cmd), $result, $returnVar);

            if ($returnVar !== 0) {
                if (stripos($result[0], 'command not found') !== false) {
                    echo "\n";
                    echo "Composer not found, you should install it.";
                    die;

                }
                echo "I cannot run composer, maybe you run '$cmd', by yourself?";
                die;
            }

            echo "...components are installed succesfully.\n";
            echo "\n";
        } catch (Exception $exception) {
            echo sprintf('Caught exception: %s', $exception->getMessage());
        }
    }
}

use Symfony\Component\Console\Application;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

require __DIR__.'/vendor/autoload.php';

$config = json_decode(file_get_contents(__DIR__ . '/composer.json'));

(new Application('echo', $config->version))
    ->register('help')
    ->setDescription('Just help')
    ->setCode(function (InputInterface $input,  OutputInterface $output) {
        $config = json_decode(file_get_contents(__DIR__ . '/composer.json'));

        $output->writeln(sprintf("Hello in %s, version: %s", $config->name, $config->version));
    })

    ->getApplication()
    ->register('require')
    ->setAliases(['req'])
    ->setDescription('Add bundles')
    ->addArgument('bundles', InputArgument::IS_ARRAY | InputArgument::REQUIRED)
    ->setCode(function (InputInterface $input, OutputInterface $output) {
        $dir = '..';

        $singleLineArguments = implode(" ", $input->getArgument('bundles'));

        exec("cd $dir && composer req --dev $singleLineArguments --no-scripts");

        $config = json_decode(file_get_contents(__DIR__. "/$dir/composer.json"));

        $packages = $config->extra->mozart->packages;

        foreach ($input->getArgument('bundles') as $bundle) {
            $bundle = explode(":", $bundle);
            $packages[] = $bundle[0];
        }

        $packages = array_unique($packages);

        $config->extra->mozart->packages = array_values($packages);

        file_put_contents(__DIR__. "/$dir/composer.json", json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES));

        exec("cd $dir && composer run-script post-install-cmd");
    })

    ->getApplication()
    ->register('remove')
    ->setAliases(['rm'])
    ->setDescription('Remove bundles')
    ->addArgument('bundles', InputArgument::IS_ARRAY | InputArgument::REQUIRED)
    ->setCode(function (InputInterface $input, OutputInterface $output) {
        $dir = '..';

        $singleLineArguments = implode(" ", $input->getArgument('bundles'));

        exec("cd $dir && composer remove --dev $singleLineArguments --no-scripts");

        $config = json_decode(file_get_contents(__DIR__. "/$dir/composer.json"));

        $packages = $config->extra->mozart->packages;

        foreach ($packages as $package)
        {

            foreach ($input->getArgument('bundles') as $bundle) {
                if ($package != $bundle) {
                    $newPackages[] = $package;
                }
            }
        }

        $packages = array_unique($newPackages);

        $config->extra->mozart->packages = array_values($packages);

        file_put_contents(__DIR__. "/$dir/composer.json", json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES));

        exec("cd $dir && composer run-script post-install-cmd");
    })

    ->getApplication()
    ->register('change')
    ->setDescription('Change namespace prefix')
    ->addArgument('prefix', InputArgument::REQUIRED)
    ->setCode(function (InputInterface $input, OutputInterface $output) {
        $prefix = $input->getArgument('prefix');
        $dir = '..';

        $config = json_decode(file_get_contents(__DIR__. "/$dir/composer.json"));

        // change main plugin prefix
        $psr4 = 'psr-4';
        $mainPrefix = $config->autoload->$psr4;

        foreach ($mainPrefix as $value) {
            $newMainPrefix = [$prefix.'\\' => $value];
        }

        $config->autoload->$psr4 = $newMainPrefix;

        // change mozart plugin/dependencies prefix
        $mozartPrefix = $config->extra->mozart->dep_namespace;
        $arr = explode("\\", $mozartPrefix);

        $arr[0] = $prefix;

        $config->extra->mozart->dep_namespace = implode("\\", $arr);

        file_put_contents(__DIR__. "/$dir/composer.json", json_encode($config, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES));

        exec("cd $dir && composer run-script post-install-cmd");
    })

    ->getApplication()
    ->setDefaultCommand('help', false)
    ->run();

