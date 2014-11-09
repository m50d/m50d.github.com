---
published: false
---

## The Difference Between Precision and Accuracy

[A year ago](https://www.chrisstucchio.com/blog/2013/basic_income_vs_basic_job.html), Chris Stucchio (AKA yummyfajitas) wrote on the subject of basic income. He wrote some code, ran some monte carlo simulations, and came out with some quite shocking results; a basic job was far, far superior to a basic income. He offered no explanation for this result, beyond the code itself, and wrote stridently that those who disagreed should offer code rather than words.

But a money gap of the size described doesn't appear out of nowhere. To understand what's going on in Stucchio's simulations, we have to do exactly what he doesn't want to: look at the numbers, and think about what they mean, in words. Fortunately, [someone has done the first step of simplification for us](https://gist.github.com/anonymous/7452196). Stucchio's original model has a bunch of epicycles that make it look more impressive than it is; the meat is here. And in fact most of the variation comes from a much simpler source: just the difference between the number of people given basic income / job:

    num_adults = 227e6
    basic_income = 7.25*40*50
    labor_force = 154e6
    disabled_adults = 21e6
    current_wealth_transfers = 3369e9
     
    def basic_income_cost_benefit():
        return num_adults * basic_income
     
    def basic_job_cost_benefit():
        return (num_adults * labor_force) * basic_income
