#!/bin/bash

target_score=$(./kube-bench master --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml --json | jq .Totals.total_fail)


if [[ $target_score -gt 10]];
then
   echo 'Failure is high please remedy the fails'
   exit 1;
else
   echo 'Test passed'
   exit 0;
fi
