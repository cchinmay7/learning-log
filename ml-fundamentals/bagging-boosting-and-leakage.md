# Bagging, Boosting, and Data Leakage

**Studied:** 13 Nov 2026
**Sources:** StatQuest: [Bias and Variance](https://www.youtube.com/watch?v=EuBBz3bI-aA), [Random Forests Part 1](https://www.youtube.com/watch?v=J4Wdy0Wc_xQ), [Gradient Boost Part 1](https://www.youtube.com/watch?v=3CC4N4z3GJc)
**Why:** I used XGBoost at work as a black box. These notes open up the box.

## Summary in 3 lines
- **Bagging** (Random Forest): many trees built *independently, in parallel* on random samples, then averaged. It mainly reduces **variance** (overfitting).
- **Boosting** (Gradient Boosting, XGBoost): trees built *one after another*, each one fitting the errors (residuals) of the trees before it. It mainly reduces **bias** (underfitting).
- **Data leakage**: training on information the model won't have at prediction time. It gives great test scores and bad real-world results.

## Bias vs. variance
- **High bias** = underfitting. The model is too simple, so it's wrong on training *and* test data.
- **High variance** = overfitting. The model memorizes training noise, so it's great on training data and bad on test data.

## Bagging / Random Forest
1. **Bootstrap**: each tree trains on a random sample of rows, drawn with replacement.
2. **Random features**: at each split, a tree only considers a random subset of features.
3. **Aggregate**: average the predictions for regression. For classification, average the class probabilities ("soft voting") and pick the highest.

**Why random features?** Without them, one strong feature (e.g., market/location) becomes every tree's first split, so the trees end up **correlated**. Averaging only helps when trees make *different* mistakes that cancel out. 500 near-identical trees are barely better than one.

**Ties?** Regression averages, so there are no ties. Classification uses probability averaging, so exact ties are rare. If one happens, the library picks the first class. A near-50/50 split means the model is unsure, and that case may be worth a human review.

## Boosting / Gradient Boosting
1. Start with a simple guess (e.g., the average target).
2. Compute the **residuals** (actual − predicted).
3. Fit a small tree to the residuals.
4. Add that tree's prediction × **learning rate** to the model.
5. Repeat.

**Why a learning rate?** Adding each tree's full prediction would fit the training data, noise included, in a few big jumps. That's low bias but high variance, i.e., overfitting. A small learning rate (e.g., 0.1) makes the model take many small steps, so no single tree dominates, and it generalizes better.
**Tradeoff:** a smaller learning rate needs more trees, which means more computation. So `learning_rate`, `n_estimators`, and **early stopping** are tuned together.

**Note:** AdaBoost is a different boosting method. It reweights the examples that were predicted wrong instead of fitting residuals. Gradient Boosting and XGBoost fit residuals.

## Data leakage
**Definition:** information in training that won't be available when the model is actually used.
**Test for every feature:** *"At the moment I'd make this prediction in real life, would I have this value?"*

| Type | Example |
|---|---|
| Time leakage | Random split on 2015–2023 deals: the model trains on 2023 and is tested on 2016, so it's "seeing the future" |
| Target leakage | Using **sale price** to predict Cap Rate (= NOI ÷ price). The model just divides, and sale price is unknown before the sale |
| Preprocessing leakage | Scaling or filling in missing values with whole-dataset statistics before splitting. Always split first |

**Warning sign:** one feature dominates the importance chart *and* the test score looks too good to be true.

## Scenario: why the test score overpromised
Cap Rate model trained on a random 80/20 split of 2015–2023 deals, then used on 2024 deals:
1. **Random split leaks the future**, so the test was too easy. **Fix:** split by time (train 2015–21, validate 2022, test 2023).
2. **Data drift**: interest rates jumped in 2022–23, which changed the relationship between property features and Cap Rates. **Fix:** add rate features, monitor, and retrain regularly.

## What I got wrong in the quiz (review these)
- Mixed up which method reduces bias vs. variance. **Bagging → variance. Boosting → bias.**
- Said the learning rate belongs to bagging. It's a **boosting** concept.
- Missed that a random split on time-based data is leakage.
- Said "no variance" when I meant "high variance."

## Questions I still have
- (Add your own)

## Next project idea
Build a property price / Cap Rate-style model on public data (e.g., NYC Rolling Sales from NYC Open Data):
- Compare Random Forest vs. XGBoost
- Use a **time-based split** and check for leakage on every feature
- Experiment with learning rate and number of trees, and plot the results
- Write up what I found in this repo
