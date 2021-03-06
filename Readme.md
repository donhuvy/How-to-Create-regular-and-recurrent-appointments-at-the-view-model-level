# How to: Create regular and recurrent appointments at the view model level

This example illustrates how to add a new regular or recurrent appointment programmatically when the **Scheduler** is in bound mode.

> **NOTE:**
> It's essential that your data source type implements the [INotifyCollectionChanged](https://docs.microsoft.com/en-us/dotnet/api/system.componentmodel.inotifypropertychanged?view=netframework-4.8) (e.g., [ObservableCollection\<T\>](https://docs.microsoft.com/en-us/dotnet/api/system.collections.objectmodel.observablecollection-1?view=netframework-4.8)). In this case, the **Scheduler Control** will receive notifications about its changes. 


To add a new appointment, create a new data item instance, define its properties, and add it to your source. In this example, SchedulerControl's [SelectedInterval](https://docs.devexpress.com/WPF/DevExpress.Xpf.Scheduling.SchedulerControl.SelectedInterval) property is bound to the **Interval** property from the view model. Its values are used in the data item's **Start** and **End** properties:

```cs
protected ApptViewModel CreateAppt(string subj, DateTime start, DateTime end, string description) {
    ApptViewModel apptViewModel = new ApptViewModel() {
        Subject = subj,
        Start = start,                
        End = end,
        Description = "[add description]"
    };
    return apptViewModel;
}
```

Set the item's **Type** property to [AppoinementType.Pattern](https://docs.devexpress.com/CoreLibraries/DevExpress.XtraScheduler.AppointmentType) and define a corresponding recurrence rule in the **RecurrenceInfo** property to create a recurrent appointment. Use [RecurrenceBuilder](https://docs.devexpress.com/WPF/DevExpress.Xpf.Scheduling.RecurrenceBuilder) to generate this rule:


```cs
[Command]
public void AddAppt(bool recurrent = false) {
    var appt = CreateAppt($"New Appt #{Appointments.Count}", Interval.Start, Interval.End, "[add description]");
            
    if(recurrent) {
        appt.Type = (int)AppointmentType.Pattern;
        appt.RecurrenceInfo = RecurrenceBuilder.Daily(Interval.Start, 10).Build().ToXml();
    } else {
        appt.Type = (int)AppointmentType.Normal;
    }
            
    this.Appointments.Add(appt);
    this.SelectedAppointments.Clear();
    this.SelectedAppointments.Add(appt);
}
```

Refer to the [How to: Create Recurrence in Code](https://docs.devexpress.com/WPF/119648/Controls-and-Libraries/Scheduler/Examples/How-to-Create-Recurrence-in-Code) article for more information about generating recurrence rules.

This example also illustrates how you can invoke EditAppointmentWindow for a newly created appointment. This functionality is implemented with the help of the [CompositeCommandBehavior](https://docs.devexpress.com/WPF/18124/mvvm-framework/behaviors/predefined-set/compositecommandbehavior) class. The first CommandItem's Command property is bound to a property from the view model. The second CommandItem's Command is bound to SchedulerControl's [Commands.ShowAppointmentWindowCommand](https://docs.devexpress.com/WPF/DevExpress.Xpf.Scheduling.SchedulerCommands.ShowAppointmentWindowCommand) command: 

```xaml
<dxb:BarButtonItem Content="Add a regular appointment">
    <dxmvvm:Interaction.Behaviors>
        <dxmvvm:CompositeCommandBehavior CanExecuteCondition="AnyCommandCanBeExecuted">
            <dxmvvm:CommandItem Command="{Binding AddApptCommand}"
                                CommandParameter="false" />
            <dxmvvm:CommandItem CheckCanExecute="False"
                                Command="{Binding ElementName=scheduler,
                                                  Path=Commands.ShowAppointmentWindowCommand}" />
        </dxmvvm:CompositeCommandBehavior>
    </dxmvvm:Interaction.Behaviors>
</dxb:BarButtonItem>
```