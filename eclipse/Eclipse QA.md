# EditorPart vs. EditPart

The main differences between an EditorPart and an EditPart in Eclipse are: [[1]](https://stackoverflow.com/questions/3259490)

- EditorPart represents an editor in the Eclipse workbench. It is responsible for editing a model object and handling save/load operations.
- EditPart on the other hand represents a part of the model being edited in the editor. It is responsible for visualizing and editing a particular part of the model.
- An EditorPart can contain multiple EditParts to represent different parts of the model being edited. So an EditorPart manages one or more EditParts.
- EditorParts are associated with file types using the IEditorInput interface. EditParts don't have direct file associations. [[2]](https://stackoverflow.com/questions/1929961)
- EditorParts handle dirty state and saving of the model. EditParts contribute to editing the model but don't directly handle saving.
- EditorParts are registered with the workbench and opened through the workbench API. EditParts are created and managed by the GraphicalViewer of the EditorPart.

So in summary, an EditorPart manages the overall editing of a model through associated EditParts which visualize and allow editing individual parts of the model. EditorParts provide the high-level editing functionality while EditParts focus on visual representation and interaction aspects.

.