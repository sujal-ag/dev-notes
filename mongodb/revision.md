# MongoDB — Queries, Populate & Aggregate Pipeline
> Dev Notes | Revision Reference

---

## 1. Normal Queries (findOne, find, findOneAndUpdate)

### find
```javascript
// get all documents
await Appointment.find();

// with filter
await Appointment.find({ userId: req.user._id });

// with field selection (projection)
await Appointment.find({ userId }, 'slotDate slotTime status');
```

### findOne
```javascript
// find single document
await Doctor.findOne({ email });

// find by id
await Doctor.findById(docId);
```

### findOneAndUpdate
```javascript
// find + update in ONE atomic operation
await Doctor.findOneAndUpdate(
    { _id: docId },          // filter
    { $set: { name: "Dr. John" } }, // update
    { new: true }            // return updated document
);
```

### Common Update Operators
| Operator | What it does | Example |
|----------|-------------|---------|
| `$set` | set a field value | `{ $set: { name: "John" } }` |
| `$unset` | remove a field | `{ $unset: { refreshToken: 1 } }` |
| `$push` | add to array | `{ $push: { slots: "10:00" } }` |
| `$pull` | remove from array | `{ $pull: { slots: "10:00" } }` |
| `$inc` | increment a field | `{ $inc: { views: 1 } }` |
| `$ne` | not equal (in filter) | `{ age: { $ne: 18 } }` |

---

## 2. Populate (Mongoose only)

Replaces a reference ID with the actual document. Essentially a **left join**.

```javascript
// basic populate
await Appointment.find({ userId }).populate('doctorId');

// populate with field selection
await Appointment.find({ userId }).populate('doctorId', 'name speciality fees');

// multiple populates
await Appointment.find({ userId })
    .populate('doctorId', 'name speciality')
    .populate('userId', 'name email');
```

### How populate works internally
```
Query 1 → fetch appointments
Query 2 → fetch doctors using $in on collected IDs
Mongoose → merges them in Node.js (application layer)
```

### Result without populate
```javascript
{ doctorId: "6993261698ea6c60e73f3bbd", slotTime: "10:00" }
//           ↑ just an ID
```

### Result with populate
```javascript
{ 
    doctorId: { name: "Dr. Richard", fees: 40, speciality: "General physician" },
    slotTime: "10:00" 
}
//  ↑ full document
```

---

## 3. Aggregate Pipeline

Runs across multiple documents in a collection. Each object in the array is a **stage**.

```javascript
await Appointment.aggregate([
    { $match: {} },    // stage 1
    { $lookup: {} },   // stage 2
    { $project: {} },  // stage 3
])
```

### Common Stages

#### $match — filter documents (like WHERE in SQL)
```javascript
{ $match: { status: 'pending', userId: userId } }
```

#### $lookup — join another collection (like LEFT JOIN in SQL)
```javascript
{ $lookup: {
    from: 'doctors',         // collection to join
    localField: 'doctorId',  // field in current collection
    foreignField: '_id',     // field in joined collection
    as: 'doctorDetails'      // output field name
}}
```

#### $project — select/reshape fields (like SELECT in SQL)
```javascript
{ $project: {
    slotTime: 1,
    slotDate: 1,
    'doctorDetails.name': 1,
    'doctorDetails.fees': 1,
    _id: 0  // exclude _id
}}
```

#### $group — group and aggregate (like GROUP BY in SQL)
```javascript
{ $group: {
    _id: '$doctorId',               // group by doctorId
    totalAppointments: { $sum: 1 }, // count
    totalRevenue: { $sum: '$amount' } // sum
}}
```

#### $sort — sort results
```javascript
{ $sort: { createdAt: -1 } } // -1 descending, 1 ascending
```

#### $limit and $skip — pagination
```javascript
{ $skip: 0 },   // skip 0 documents (page 1)
{ $limit: 10 }  // return 10 documents
```

### Real Example — Doctor Dashboard
```javascript
// get all appointments for a doctor with patient details
await Appointment.aggregate([
    { $match: { doctorId: new mongoose.Types.ObjectId(doctorId) } },
    { $lookup: {
        from: 'users',
        localField: 'userId',
        foreignField: '_id',
        as: 'patient'
    }},
    { $unwind: '$patient' }, // flatten array to object
    { $project: {
        slotDate: 1,
        slotTime: 1,
        status: 1,
        amount: 1,
        'patient.name': 1,
        'patient.email': 1,
    }},
    { $sort: { createdAt: -1 } }
])
```

### Real Example — Admin Dashboard Stats
```javascript
// total revenue and appointments per doctor
await Appointment.aggregate([
    { $match: { payment: true } },
    { $group: {
        _id: '$doctorId',
        totalAppointments: { $sum: 1 },
        totalRevenue: { $sum: '$amount' }
    }},
    { $lookup: {
        from: 'doctors',
        localField: '_id',
        foreignField: '_id',
        as: 'doctor'
    }},
    { $sort: { totalRevenue: -1 } }
])
```

---

## 4. Populate vs $lookup

| | Populate | $lookup |
|---|---|---|
| Queries | 2 separate queries | 1 single query |
| Joining happens | Node.js (app layer) | MongoDB (DB level) |
| Performance | Slower for large data | Faster |
| Syntax | Simpler | More verbose |
| Use when | Simple joins, small data | Complex joins, large data |

```
populate  → 2 round trips to DB → Node.js merges
$lookup   → 1 round trip to DB  → DB merges
```

**Rule of thumb:**
- Simple appointment + doctor details → use `populate`
- Dashboard stats, multiple joins, filtering after join → use `$lookup`

---

## 5. Atomicity

| Operation | Atomic? |
|-----------|---------|
| `findOneAndUpdate` | ✅ Yes — check + update in one operation |
| `populate` | ❌ No — 2 separate queries |
| `aggregate` | ❌ Not by default — reads only |
| Transaction | ✅ Yes — multiple operations guaranteed |

### Atomic booking example
```javascript
// check slot is free + book it = ONE atomic operation
const doctor = await Doctor.findOneAndUpdate(
    {
        _id: docId,
        [`slot_booked.${slotDate}`]: { $ne: slotTime } // slot NOT booked
    },
    {
        $push: { [`slot_booked.${slotDate}`]: slotTime } // push slot
    },
    { new: true }
);

if (!doctor) throw new ApiError(409, "Slot already booked");
// if null → slot was taken → 409
// if doctor → slot was free → booked ✅
```

---

## 6. Transactions (multiple atomic operations)

Use when multiple operations must ALL succeed or ALL fail together.

```javascript
const session = await mongoose.startSession();
session.startTransaction();

try {
    await Doctor.findOneAndUpdate(..., { session });
    await Appointment.create([{...}], { session });
    
    await session.commitTransaction(); // both succeed
} catch (error) {
    await session.abortTransaction();  // both rollback
} finally {
    session.endSession();
}
```

---

*Last updated: Feb 2026*