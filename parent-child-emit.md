# How Child talk to parent

## Understanding so far

Parent -> (subscribe) -> topic <- (publish) <- Child

Pesudo code now, so we start to define topic name first inside parent level:

In parent component:

```
@topicName="fnToReceiveChildProps"

fnToReceiveChildProps(childProps) {
  console.log("reading child level props/values", childProps);
}
```

Now turn to the child level, which publishes the props/values to parent level.

In child component:

```
<input type="submit" @click="$emit('topicName', { prop: 'prop', value: 'value' })" />
```

Thast it !!

Always remember this formula, better for writing Vue components more, more practice, solid memo then ~
