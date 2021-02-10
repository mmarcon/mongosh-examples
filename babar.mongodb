const barbar = require('barbar');
use('maxtest_fle');
const results = db.customers.aggregate([{
  $group: {
    _id: '$birthYear',
    numberOfCustomers: {
      $sum: 1
    }
  }
}, {
  $sort: {
    numberOfCustomers: -1
  }
}]).toArray().map(({_id, numberOfCustomers}) => [_id, numberOfCustomers]);