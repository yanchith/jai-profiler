# A minimal intrusive profiler for JAI

This is a small intrusive profiler module for the JAI language based on the
[design](https://github.com/cmuratori/computer_enhance/blob/main/perfaware/part2/listing_0096_selectable_profiler.cpp)
presented by Casey Muratori in 2023 in his [Computer, Enhance!](https://www.computerenhance.com/)
programming educational series. (This is in term inspired by earlier works of Sean Barett and
Jonathan Blow.)

The profiler measures inclusive and exclusive time of each profiled zone, as well as the number of
times the zone was entered. It does not do call attribution.

The cool thing about this implementation is that because of the metaprogramming available in JAI,
the runtime only does the minimum amount of work necessary for each profiling zone, which is one
less write compared to the original. The overhead for each profiling zone is two timestamp
measurements (e.g. rdtsc on x64), a couple of loads and stores to one cache line, and a few adds and
subtracts.

## Usage

Drop the Profiler directory into your modules.

Compile with: `jai my_thing.jai +Profiler`. (If you compile without the metaprogram plugin, every profiler entry point becomes a no-op.)

```jai
#import "Basic";
#import "Profiler"()(IMPORT_MODE = .RUNTIME); // You can import into multiple modules, in which case you only specify the program parameter once.

main :: () {
    // Call profiler_init at the start of your program
    profiler_init();

    while true {
        // Start each profiling cycle (for instance a frame in a video game) with profiler_start_frame
        profiler_start_frame();

        // Call your stuff
        my_frame();

        // Call profiler_end_frame at the end of the code you want to profile. This gives
        // you the scale with which to multiply the timing data in the profiling zones.
        time_scale := profiler_end_frame();

        // Get the zones from the profiler for display (in this case sorted by inclusive time)
        zones, zone_labels := profiler_get_zones(.INCLUSIVE_TIME,, temp);

        // ... Display the profiling information for the frame ...

        reset_temporary_storage();
    }
}

my_frame :: () {
    // Mark blocks you want timed with this macro. It times everything from the invocation until the end of scope.
    ProfileZone("my_frame");
    x: [..] s64;

    {
        ProfileZone("a"); // You can time a more focused part of the code by using blocks...
        for 0..9999 {
            array_add(*x, it);
        }
    }

    sum := 0;
    for x {
        ProfileZone("b"); // ... or any imperative syntax that introduces scopes, like for, if, while, defer, case, etc.
        sum += it;
    }
}
```

Also see the example at: [./Profiler/example.jai](./Profiler/example.jai).
