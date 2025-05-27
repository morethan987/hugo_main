---
title: Bayesian Optimization
weight: -10
draft: false
description: Principles of Bayesian Optimization and its MATLAB Implementation
slug: bayesianopt
tags:
  - math
  - MATLAB
  - CUMCM
series:
  - MathModel
series_order: 1
date: 2024-08-05
lastmod: 2024-12-20
authors:
  - Morethan
---

## References  

My understanding of Bayesian optimization is also limited, mainly referencing the following content👇  

- [【Machine Learning】Understanding Bayesian Optimization in One Article](https://blog.csdn.net/qq_27590277/article/details/115451660)  

- [A Detailed Explanation of Bayesian Optimization Principles](https://www.cnblogs.com/milliele/p/17782631.html)  

- [Bayesian Optimization](https://blog.csdn.net/Leon_winter/article/details/86604553)  

- [Hyperparameter Optimization—Bayesian Optimization and Its Improvement (PBT Optimization)](https://blog.csdn.net/xys430381_1/article/details/103871212)  

- [Bayesian Optimization](https://leovan.me/cn/2020/06/bayesian-optimization/)  

- [MATLAB Official Documentation](https://ww2.mathworks.cn/help/stats/bayesopt.html?s_tid=srchtitle_site_search_1_bayesopt)  

- [Exploring Bayesian Optimization](https://distill.pub/2020/bayesian-optimization/)  

## Advantages and Algorithm Principles  

This section focuses on the advantages of Bayesian optimization and its algorithmic principles. If you are only interested in "how to use it," you can first learn about the advantages of Bayesian optimization and then skip to [ MATLAB Usage]({{< relref "#matlab-usage" >}}).  

### Advantages  

### Algorithm Principles  

## MATLAB Usage  

### Code Overview  

```matlab  
% Define the objective function  
function y = objectiveFcn(x)  
    y = (1 - x.x1)^2 + 100 * (x.x2 - x.x1^2)^2;  
end  

% Define optimization variables  
vars = [optimizableVariable('x1', [-2, 2])  
        optimizableVariable('x2', [-2, 2])];  

% Perform Bayesian optimization  
results = bayesopt(@objectiveFcn, vars, ...  
                   'AcquisitionFunctionName', 'expected-improvement-plus', ...  
                   'MaxObjectiveEvaluations', 30, ...  
                   'IsObjectiveDeterministic', true, ...  
                   'Verbose', 1);  

% View results  
bestPoint = results.XAtMinObjective;  
bestObjective = results.MinObjective;  

fprintf('Optimal solution x1: %.4f, x2: %.4f\n', bestPoint.x1, bestPoint.x2);  
fprintf('Optimal objective value: %.4f\n', bestObjective);  
```  

### Parameter Explanation  

| Params                     | Meaning                                     |  
| -------------------------- | ------------------------------------------- |  
| `AcquisitionFunctionName`  | Selects the acquisition function, which determines how the algorithm chooses the next sampling point after each iteration. |  
| `MaxObjectiveEvaluations`  | Maximum number of iterations.               |  
| `IsObjectiveDeterministic` | Set to `true` if the objective function is deterministic (no noise); otherwise, set to `false`. |  
| `Verbose`                  | Controls the verbosity of the output, which may include multiple charts. |  

For specific options for each parameter, refer to the official documentation: [bayesopt](https://ww2.mathworks.cn/help/stats/bayesopt.html?s_tid=srchtitle_site_search_1_bayesopt). The official documentation is very detailed and includes many examples.  


{{< alert icon="pencil" cardColor="#1E3A8A" textColor="#E0E7FF" >}}
One of the essential skills for mathematical modelers is reading documentation 😝
{{< /alert >}}

