---
layout: code-snippet
title:  "Model finder"
date:   2018-03-14
excerpt: ""
tag:
- snippet
---
As a developer, I'm always trying to automate anything I can. You may call me lazy but I prefer to think of it as being clever. This desire to automate as much as I can appeared when I was working on a machine learning project a while ago. I realized that out-of-the-box algorithms (linear models, SVM, random forest, a basic neural net etc.) worked pretty well for a lot of cases, even for situations where they didn't it was good to use them as bench marks. I always munge my dataset into a tidy format, which, would be very hard to automate. After this step however, I saw an opportunity to automate the training of different models mentioned above. I ended up creating an object-oriented program called model-finder that takes in a dataset and evaluates the performance of several different machine learning models. This program has two main abstractions, a dataset object and a model finder object. This approach to machine learning is called automated machine learning. Utilizing this process really helps me to make the model generation process more reproducible and stable for use in production code. Below is a code snippet for the dataset abstraction portion of this program. 

{% highlight py %}
class DataSet(object):
    def __init__(self, dataset_path, drop_features_ls, target_col, groupby_col, validation_size, test_size, k_folds, output_path, debug=False):
        random.seed(10)
        self.data = pd.read_csv(dataset_path, sep=',')
        self.name = self._make_filename(dataset_path, drop_features_ls)
        self._check_directories()
        self.drop_features_ls = drop_features_ls
        self.target_col = target_col
        self.groupby_col = groupby_col
        self.val_size = float(val_size)
        self.test_size = float(test_size)
        self.k_folds = int(k_folds)
        self.output_path = output_path
        self.debug = debug
        self.train_features = None
        self.train_targets = None
        self.test_features = None
        self.test_targets = None

    def _make_filename(self, filepath, drop_features_ls):
        drop_features_ls = [drop_features_ls] if type(drop_features_ls) is not list else drop_features_ls
        filename = filepath.split('/')[-1].split('.')[0]
        features = '_'.join(drop_features_ls) if drop_features_ls is not None else drop_features_ls
        if features:
            self.name = '{}_drop_{}'.format(filename, features)
        else:
            self.name = filename
        return self.name

    def _check_directories(self):
        if not os.path.exists('results'):
            os.makedirs('results')
        if not os.path.exists('results/{}'.format(self.name)):
            os.makedirs('results/{}'.format(self.name))
        return self

    def _remove_features(self, dataset):
        if self.drop_features_ls is not None:
            return dataset.drop(columns=self.drop_features_ls, axis=1)
        else:
            return dataset

    def _k_folds_generator(self, folds_dict):
        for i in xrange(len(folds_dict.keys())):
            test_fold = 'fold_{}'.format(i + 1)
            test_set = folds_dict[test_fold]
            train_list = (fold for fold in folds_dict.keys() if fold not in test_fold)
            train_frames = []
            train_frames.extend(map(lambda x: folds_dict[x], train_list))
            train_set = pd.concat(train_frames)
            yield test_set, train_set

    def create_val_set(self, dat=None, groupby_col=None, valset_size=None):
        dat = self.data if dat is None else dat
        groupby_col = self.val_groupby if groupby_col is None else groupby_col
        valset_size = self.val_size if valset_size is None else valset_size
        folder = '{}_validation_set_{}_rows'.format(self.name, valset_size)

        if groupby_col is not None:
            unique_ls = dat[groupby_col].unique().tolist()
            val_ls = random.sample(unique_ls, int(len(unique_ls) * valset_size))
            self.data = dat[~dat[groupby_col].isin(val_ls)]
            val_set = dat[dat[groupby_col].isin(val_ls)]
        else:
            self.data, val_set = train_test_split(dat, shuffle=True, test_size=valset_size)


        features, targets = self.vectorize(val_set, self.target_col)

        self.write_dataset(folder, 'training_data', self.data)
        self.write_dataset(folder, 'validation_set_full', val_set)
        self.write_dataset(folder, 'features_validation_set', features)
        self.write_dataset(folder, 'targets_validation_set', targets)

        if self.debug:
            print('validation set is {} long and data is now {}'.format(val_set.shape, self.data.shape))
        return self

    def split_into_k_folds(self, dat=None, groupby_col=None, k_folds=None):
        dat = self.data if dat is None else dat
        k_folds = self.k_folds if k_folds is None else k_folds
        groupby_col = self.test_train_groupby if groupby_col is None else groupby_col
        folder = '{}_K-fold_{}_groupedby_{}'.format(self.name, k_folds, groupby_col)

        df_split = np.array_split(dat, k_folds)
        fold_dict = {'fold_{}'.format(i+1): v for i, v in enumerate(df_split)}

        try:
            fold_generator = self._k_folds_generator(fold_dict)
            for i in xrange(len(fold_dict.keys())):
                k_folder = '{}/K_{}'.format(folder, i+1)
                test, train = next(fold_generator)

                train_features, train_targets = self.vectorize(train, self.target_col)
                test_features, test_targets = self.vectorize(test, self.target_col)

                setattr(self, train_features, train_features)
		        setattr(self, train_targets, train_targets)
		        setattr(self, test_features, test_features)
		        setattr(self, test_targets, test_targets)

        except StopIteration:
            return self

        if self.debug:
            print('{} folds created'.format(k_folds))
        return self

    def split_into_test_and_train(self, name=None, dat=None, test_size=None, groupby_col=None):
        dat = self.data if dat is None else dat
        test_size = self.test_size if test_size is None else test_size
        groupby_col = self.test_train_groupby if groupby_col is None else groupby_col
        name = self.name if name is None else name
        folder = '{}_test_train_split_{}_groupedby_{}'.format(name, test_size, groupby_col)

        train, test = train_test_split(dat, shuffle=True, test_size=test_size)

        train_features, train_targets = self.vectorize(train, self.target_col)
        test_features, test_targets = self.vectorize(test, self.target_col)

        setattr(self, train_features, train_features)
        setattr(self, train_targets, train_targets)
        setattr(self, test_features, test_features)
        setattr(self, test_targets, test_targets)

        if self.debug:
            print('train ({} rows) test ({} rows) split was created'.format(train.shape, test.shape))
        return selfs

    def vectorize(self, dataset, target_col):
        scaler = preprocessing.MinMaxScaler()
        dataset = self._remove_features(dataset)
        targets = dataset[[target_col]]
        features = dataset.drop(columns=target_col, axis=1)
        feature_cols = features.columns.tolist()
        scaled_features = scaler.fit_transform(features)
        scaled_features = pd.DataFrame(scaled_features, columns=feature_cols)
        return scaled_features, targets

    def one_hot_encode(self, dat, columns):
        dat = self.data if dat is None else dat
        columns = columns if type(columns) is list else list(columns)
        return pd.get_dummies(dat, columns=columns)

    def write_dataset(self, folder, filename, dataset):
        if not os.path.exists('results/{}/{}'.format(self.name, folder)):
            os.makedirs('results/{}/{}'.format(self.name, folder))
        dataset.to_csv('results/{}/{}/{}.gz'.format(self.name, folder, filename), sep=',', compression='gzip', index=False)
        return self

# sample usage would be
dat = DataSet('path/to/file.csv', ['features', 'to', 'drop'], 'prediction_col', None, 0.2, 0.2, 5, '.', debug=False)
dat.create_val_set()

# Then we would plug it into the finder object that would utilize the splitting/vectorizing methods as necessary 
model_grid = Finder(dat, type='classification')
model_grid.find_model(test_metrics=[roc_auc_score])
{% endhighlight %}
