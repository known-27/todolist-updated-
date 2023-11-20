# todolist-updated-


import javax.swing.*;
import javax.swing.undo.*;
import java.awt.*;
import java.awt.event.*;
import java.time.LocalDateTime;
import java.util.ArrayList;

class Task {
  private String description;
  private LocalDateTime dateTime;

  public Task(String description, LocalDateTime dateTime) {
    this.description = description;
    this.dateTime = dateTime;
  }

  public String getDescription() {
    return description;
  }

  public LocalDateTime getDateTime() {
    return dateTime;
  }

  @Override
  public String toString() {
    return description + " (Due: " + dateTime.toString() + ")";
  }
}

public class todolist2 {
  private static ArrayList<Task> toDoList = new ArrayList<>();
  private static DefaultListModel<Task> listModel;
  private static JList<Task> list;
  private static UndoManager undoManager = new UndoManager();

  public static void main(String[] args) {
    JFrame frame = new JFrame("To-Do List App");
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.setSize(600, 300);
    frame.setExtendedState(JFrame.MAXIMIZED_BOTH); // Maximize the window

    JPanel panel = new JPanel();
    panel.setLayout(new BorderLayout());

    listModel = new DefaultListModel<>();
    list = new JList<>(listModel);
    JScrollPane scrollPane = new JScrollPane(list);
    scrollPane.setBorder(BorderFactory.createEmptyBorder()); // Remove the border
    panel.add(scrollPane, BorderLayout.CENTER);

    JList<Task> numberedTaskList = new JList<>(new AbstractListModel<Task>() {
      @Override
      public int getSize() {
        return toDoList.size();
      }

      @Override
      public Task getElementAt(int index) {
        return toDoList.get(index);
      }
    });

    panel.add(new JScrollPane(numberedTaskList), BorderLayout.WEST);

    JPanel optionsPanel = new JPanel();
    optionsPanel.setLayout(new FlowLayout());

    JTextField taskField = new JTextField(20);

    JButton addButton = new JButton("Add Task");
    JButton deleteButton = new JButton("Delete Task");
    JButton editButton = new JButton("Edit Task");
    JButton undoButton = new JButton("Undo");
    JButton redoButton = new JButton("Redo");

    addButton.addActionListener(new ActionListener() {
      @Override
      public void actionPerformed(ActionEvent e) {
        String taskDescription = taskField.getText();
        if (!taskDescription.isEmpty()) {
          LocalDateTime currentDateTime = LocalDateTime.now();
          Task task = new Task(taskDescription, currentDateTime);
          toDoList.add(task);
          listModel.addElement(task);
          taskField.setText("");
          undoManager.addEdit(new AbstractUndoableEdit() {
            @Override
            public void undo() throws CannotUndoException {
              super.undo();
              toDoList.remove(task);
              listModel.removeElement(task);
            }

            @Override
            public void redo() throws CannotRedoException {
              super.redo();
              toDoList.add(task);
              listModel.addElement(task);
            }
          });
        }
      }
    });

    deleteButton.addActionListener(new ActionListener() {
      @Override
      public void actionPerformed(ActionEvent e) {
        int selectedIndex = list.getSelectedIndex();
        if (selectedIndex != -1) {
          Task removedTask = listModel.getElementAt(selectedIndex);

          int confirmation = JOptionPane.showConfirmDialog(null, "Are you sure you want to delete this task?",
              "Confirmation", JOptionPane.YES_NO_OPTION);

          if (confirmation == JOptionPane.YES_OPTION) {
            toDoList.remove(removedTask);
            listModel.remove(selectedIndex);
            undoManager.addEdit(new AbstractUndoableEdit() {
              @Override
              public void undo() throws CannotUndoException {
                super.undo();
                toDoList.add(selectedIndex, removedTask);
                listModel.add(selectedIndex, removedTask);
              }

              @Override
              public void redo() throws CannotRedoException {
                super.redo();
                toDoList.remove(removedTask);
                listModel.removeElement(removedTask);
              }
            });
          }
        }
      }
    });

    editButton.addActionListener(new ActionListener() {
      @Override
      public void actionPerformed(ActionEvent e) {
        int selectedIndex = list.getSelectedIndex();
        if (selectedIndex != -1) {
          Task selectedTask = listModel.getElementAt(selectedIndex);
          String editedTaskDescription = JOptionPane.showInputDialog(null, "Edit Task", selectedTask.getDescription());
          if (editedTaskDescription != null && !editedTaskDescription.isEmpty()) {
            Task newTask = new Task(editedTaskDescription, selectedTask.getDateTime());
            toDoList.set(selectedIndex, newTask);
            listModel.set(selectedIndex, newTask);
            undoManager.addEdit(new AbstractUndoableEdit() {
              @Override
              public void undo() throws CannotUndoException {
                super.undo();
                toDoList.set(selectedIndex, selectedTask);
                listModel.set(selectedIndex, selectedTask);
              }

              @Override
              public void redo() throws CannotRedoException {
                super.redo();
                toDoList.set(selectedIndex, newTask);
                listModel.set(selectedIndex, newTask);
              }
            });
          }
        }
      }
    });

    undoButton.addActionListener(new ActionListener() {
      @Override
      public void actionPerformed(ActionEvent e) {
        if (undoManager.canUndo()) {
          undoManager.undo();
        }
      }
    });

    redoButton.addActionListener(new ActionListener() {
      @Override
      public void actionPerformed(ActionEvent e) {
        if (undoManager.canRedo()) {
          undoManager.redo();
        }
      }
    });

    KeyAdapter keyListener = new KeyAdapter() {
      @Override
      public void keyPressed(KeyEvent e) {
        String taskDescription = taskField.getText();
        if (e.getKeyCode() == KeyEvent.VK_ENTER) {
          if (!taskDescription.isEmpty()) {
            LocalDateTime currentDateTime = LocalDateTime.now();
            Task task = new Task(taskDescription, currentDateTime);
            toDoList.add(task);
            listModel.addElement(task);
            taskField.setText("");
            undoManager.addEdit(new AbstractUndoableEdit() {
              @Override
              public void undo() throws CannotUndoException {
                super.undo();
                toDoList.remove(task);
                listModel.removeElement(task);
              }

              @Override
              public void redo() throws CannotRedoException {
                super.redo();
                toDoList.add(task);
                listModel.addElement(task);
              }
            });
          }
        }
      }
    };

    taskField.addKeyListener(keyListener);
    list.addKeyListener(keyListener);

    optionsPanel.add(taskField);
    optionsPanel.add(addButton);
    optionsPanel.add(deleteButton);
    optionsPanel.add(editButton);
    optionsPanel.add(undoButton);
    optionsPanel.add(redoButton);

    panel.add(optionsPanel, BorderLayout.SOUTH);

    frame.getContentPane().add(panel);
    frame.setVisible(true);
  }
}
