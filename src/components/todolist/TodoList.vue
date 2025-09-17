<template>
  <div class="w-full bg-slate-900 min-h-screen">
    <!-- Tab Navigation -->
    <div class="bg-slate-800 border-b border-slate-700">
      <div class="flex items-center">
        <button
          @click="activeTab = 'maintable'"
          :class="[
            'flex items-center gap-2 px-4 py-3 text-sm font-medium transition-colors',
            activeTab === 'maintable'
              ? 'bg-slate-700 text-white border-b-2 border-blue-500'
              : 'text-slate-300 hover:text-white hover:bg-slate-700/50',
          ]"
        >
          <Home class="w-4 h-4" />
          Main Table
        </button>

        <button
          @click="activeTab = 'kanban'"
          :class="[
            'flex items-center gap-2 px-4 py-3 text-sm font-medium transition-colors',
            activeTab === 'kanban'
              ? 'bg-slate-700 text-white border-b-2 border-blue-500'
              : 'text-slate-300 hover:text-white hover:bg-slate-700/50',
          ]"
        >
          Kanban
        </button>

        <!-- Add new tab button -->
        <button
          class="p-3 text-slate-400 hover:text-white hover:bg-slate-700/50 transition-colors"
        >
          <Plus class="w-4 h-4" />
        </button>
      </div>
    </div>

    <!-- Tab Content -->
    <div class="p-6">
      <!-- Main Table -->
      <div v-if="activeTab === 'maintable'" class="text-white">
        <!-- Header Controls -->
        <div class="flex items-center gap-4 mb-6">
          <Button @click="addNewTask" class="bg-blue-600 hover:bg-blue-700">
            New task
          </Button>
          <div class="relative">
            <Search
              class="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-gray-400"
            />
            <Input
              v-model="searchQuery"
              placeholder="Search tasks..."
              class="pl-10 bg-slate-800 border-slate-700 text-white placeholder-gray-400"
            />
          </div>

          <DropdownMenu>
            <DropdownMenuTrigger as-child>
              <Button
                variant="outline"
                class="border-slate-700 text-gray-400 bg-slate-800 hover:bg-slate-700 focus:ring-2 focus:ring-slate-500"
              >
                <User class="mr-2 h-4 w-4" />
                Person
                <span v-if="selectedDevelopers.length > 0" class="ml-1">
                  ({{ selectedDevelopers.length }})
                </span>
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent
              class="bg-slate-800 border border-slate-700 rounded-md shadow-lg max-h-60 overflow-y-auto w-64"
            >
              <!-- Header dengan tombol clear -->
              <div
                class="flex items-center justify-between p-3 border-b border-slate-700"
              >
                <span class="text-gray-400 text-sm">Filter by Developer</span>
                <Button
                  v-if="selectedDevelopers.length > 0"
                  @click.stop="clearAllDevelopers"
                  variant="ghost"
                  size="sm"
                  class="text-red-400 hover:text-red-300 hover:bg-slate-700 h-7 px-2"
                >
                  Clear all
                  <X class="h-3 w-3 ml-1" />
                </Button>
              </div>

              <!-- List developers -->
              <div
                v-for="dev in uniqueDevelopers"
                :key="dev"
                @click="toggleDeveloper(dev)"
                :class="[
                  'flex items-center px-4 py-2 hover:bg-slate-700 transition-colors cursor-pointer',
                  selectedDevelopers.includes(dev) ? 'bg-slate-600' : '',
                ]"
              >
                <!-- <Checkbox
                  :checked="selectedDevelopers.includes(dev)"
                  class="mr-3 border-slate-600"
                /> -->
                <span class="text-gray-400">{{ dev }}</span>
              </div>

              <!-- Empty state -->
              <div
                v-if="uniqueDevelopers.length === 0"
                class="text-gray-500 px-4 py-3 text-sm text-center"
              >
                No developers available
              </div>
            </DropdownMenuContent>
          </DropdownMenu>

          <DropdownMenu>
            <DropdownMenuTrigger as-child>
              <Button
                variant="outline"
                class="border-slate-700 text-gray-400 bg-slate-800 hover:bg-slate-700 focus:ring-2 focus:ring-slate-500"
              >
                <ArrowUpDown class="mr-2 h-4 w-4" />
                Sort
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent
              class="bg-slate-800 border border-slate-700 rounded-md shadow-lg"
            >
              <DropdownMenuItem
                @click="setSorting('task')"
                class="text-gray-400 hover:bg-slate-700"
              >
                Sort by Task
              </DropdownMenuItem>
              <DropdownMenuItem
                @click="setSorting('priority')"
                class="text-gray-400 hover:bg-slate-700"
              >
                Sort by Priority
              </DropdownMenuItem>
              <DropdownMenuItem
                @click="setSorting('status')"
                class="text-gray-400 hover:bg-slate-700"
              >
                Sort by Status
              </DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>
        </div>
        <Accordion type="single" collapsible :default-value="'item-1'">
          <AccordionItem value="item-1">
            <AccordionTrigger class="w-auto justify-start"
              >All Task</AccordionTrigger
            >
            <AccordionContent>
              <!-- Task Table -->
              <div
                v-if="showAllTasks"
                class="rounded-lg border border-slate-700 overflow-hidden"
              >
                <Table class="bg-slate-800">
                  <TableHeader>
                    <TableRow class="border-slate-700 hover:bg-slate-700">
                      <TableHead class="text-gray-300">
                        <Checkbox
                          :checked="allSelected"
                          :indeterminate="allIndeterminate"
                          @update:checked="toggleAllSelection"
                          class="border-slate-600"
                          aria-label="Select all tasks"
                        />
                      </TableHead>
                      <TableHead class="text-gray-300">Task</TableHead>
                      <TableHead class="text-gray-300"></TableHead>
                      <TableHead class="text-gray-300">Developer</TableHead>
                      <TableHead class="text-gray-300">Status</TableHead>
                      <TableHead class="text-gray-300">Priority</TableHead>
                      <TableHead class="text-gray-300">Type</TableHead>
                      <TableHead class="text-gray-300">Date</TableHead>
                      <TableHead class="text-gray-300">Estimated SP</TableHead>
                      <TableHead class="text-gray-300">Actual SP</TableHead>
                      <TableHead class="text-gray-300">
                        <Button
                          @click="addTask"
                          size="sm"
                          variant="ghost"
                          class="text-gray-400 hover:text-white"
                        >
                          <Plus class="h-4 w-4" /> </Button
                      ></TableHead>
                    </TableRow>
                  </TableHeader>
                  <TableBody>
                    <!-- Table Row -->
                    <TableRow
                      v-for="task in filteredTasks"
                      :key="task.id"
                      class="border-slate-700 hover:bg-slate-700"
                    >
                      <TableCell>
                        <Checkbox
                          :checked="selectedTasks.includes(task.id)"
                          @update:checked="(checked: boolean) => toggleTaskSelection(task.id, checked)"
                          class="border-slate-600"
                        />
                      </TableCell>
                      <TableCell>
                        <Input
                          v-if="
                            editingTask === task.id && editingField === 'task'
                          "
                          v-model="task.task"
                          @blur="stopEditing"
                          @keyup.enter="stopEditing"
                          class="bg-slate-700 border-slate-600 text-white"
                          autofocus
                        />
                        <span
                          v-else
                          @click="startEditing(task.id, 'task')"
                          class="cursor-pointer hover:bg-slate-700 px-2 py-1 rounded"
                        >
                          {{ task.task }}
                        </span>
                      </TableCell>
                      <TableCell>
                        <MessageCirclePlus />
                      </TableCell>
                      <TableCell>
                        <TooltipProvider>
                          <div class="flex -space-x-2">
                            <!-- 3 Developer pertama -->
                            <div
                              v-for="(dev, index) in task.developer.slice(0, 3)"
                              :key="dev"
                              class="w-8 h-8 rounded-full bg-gradient-to-br from-blue-400 to-purple-500 flex items-center justify-center text-xs font-medium border-2 border-white relative z-10"
                              :style="{ zIndex: task.developer.length - index }"
                            >
                              <Tooltip>
                                <TooltipTrigger as-child>
                                  <div
                                    class="w-full h-full flex items-center justify-center cursor-pointer"
                                  >
                                    {{ dev.charAt(0).toUpperCase() }}
                                  </div>
                                </TooltipTrigger>
                                <TooltipContent
                                  class="bg-slate-800 border-slate-700 text-white"
                                >
                                  <p>{{ dev }}</p>
                                </TooltipContent>
                              </Tooltip>
                            </div>

                            <!-- Jika lebih dari 3 developer -->
                            <div
                              v-if="task.developer.length > 3"
                              class="w-8 h-8 rounded-full bg-gray-500 flex items-center justify-center text-xs font-medium text-white border-2 border-white relative"
                              :style="{ zIndex: 0 }"
                            >
                              <Tooltip>
                                <TooltipTrigger as-child>
                                  <div
                                    class="w-full h-full flex items-center justify-center cursor-pointer"
                                  >
                                    +{{ task.developer.length - 3 }}
                                  </div>
                                </TooltipTrigger>
                                <TooltipContent
                                  class="bg-slate-800 border-slate-700 text-white max-w-xs"
                                >
                                  <p class="font-medium mb-1">
                                    Other developers:
                                  </p>
                                  <div class="space-y-1">
                                    <p
                                      v-for="otherDev in task.developer.slice(
                                        3
                                      )"
                                      :key="otherDev"
                                      class="text-sm"
                                    >
                                      {{ otherDev }}
                                    </p>
                                  </div>
                                </TooltipContent>
                              </Tooltip>
                            </div>
                          </div>
                        </TooltipProvider>
                      </TableCell>
                      <TableCell>
                        <Select
                          v-if="
                            editingTask === task.id && editingField === 'status'
                          "
                          v-model="task.status"
                          @update:model-value="stopEditing"
                        >
                          <SelectTrigger class="bg-slate-700 border-slate-600">
                            <SelectValue />
                          </SelectTrigger>
                          <SelectContent class="bg-slate-800 border-slate-700">
                            <SelectItem
                              v-for="status in statusOptions"
                              :key="status"
                              :value="status"
                              class="text-gray-400"
                            >
                              {{ status }}
                            </SelectItem>
                          </SelectContent>
                        </Select>
                        <Badge
                          v-else
                          @click="startEditing(task.id, 'status')"
                          :class="getStatusColor(task.status)"
                          class="cursor-pointer"
                        >
                          {{ task.status }}
                        </Badge>
                      </TableCell>
                      <TableCell>
                        <Select
                          v-if="
                            editingTask === task.id &&
                            editingField === 'priority'
                          "
                          v-model="task.priority"
                          @update:model-value="stopEditing"
                        >
                          <SelectTrigger class="bg-slate-700 border-slate-600">
                            <SelectValue />
                          </SelectTrigger>
                          <SelectContent class="bg-slate-800 border-slate-700">
                            <SelectItem
                              v-for="priority in priorityOptions"
                              :key="priority"
                              :value="priority"
                              class="text-gray-400"
                            >
                              {{ priority }}
                            </SelectItem>
                          </SelectContent>
                        </Select>
                        <Badge
                          v-else
                          @click="startEditing(task.id, 'priority')"
                          :class="getPriorityColor(task.priority)"
                          class="cursor-pointer"
                        >
                          {{ task.priority }}
                        </Badge>
                      </TableCell>
                      <TableCell>
                        <Select
                          v-if="
                            editingTask === task.id && editingField === 'type'
                          "
                          v-model="task.type"
                          @update:model-value="stopEditing"
                        >
                          <SelectTrigger class="bg-slate-700 border-slate-600">
                            <SelectValue />
                          </SelectTrigger>
                          <SelectContent class="bg-slate-800 border-slate-700">
                            <SelectItem
                              v-for="type in typeOptions"
                              :key="type"
                              :value="type"
                              class="text-gray-400"
                            >
                              {{ type }}
                            </SelectItem>
                          </SelectContent>
                        </Select>
                        <Badge
                          v-else
                          @click="startEditing(task.id, 'type')"
                          :class="getTypeColor(task.type)"
                          class="cursor-pointer"
                        >
                          {{ task.type }}
                        </Badge>
                      </TableCell>
                      <TableCell>
                        <Input
                          v-if="
                            editingTask === task.id && editingField === 'date'
                          "
                          v-model="task.date"
                          type="date"
                          @blur="stopEditing"
                          @keyup.enter="stopEditing"
                          class="bg-slate-700 border-slate-600 text-white"
                          autofocus
                        />
                        <span
                          v-else
                          @click="startEditing(task.id, 'date')"
                          class="cursor-pointer hover:bg-slate-700 px-2 py-1 rounded"
                        >
                          {{ formatDate(task.date) }}
                        </span>
                      </TableCell>
                      <TableCell>
                        <Input
                          v-if="
                            editingTask === task.id &&
                            editingField === 'estimatedSP'
                          "
                          v-model.number="task.estimatedSP"
                          type="number"
                          @blur="stopEditing"
                          @keyup.enter="stopEditing"
                          class="bg-slate-700 border-slate-600 text-white w-20"
                          autofocus
                        />
                        <span
                          v-else
                          @click="startEditing(task.id, 'estimatedSP')"
                          class="cursor-pointer hover:bg-slate-700 px-2 py-1 rounded"
                        >
                          {{ task.estimatedSP }} SP
                        </span>
                      </TableCell>
                      <TableCell>
                        <Input
                          v-if="
                            editingTask === task.id &&
                            editingField === 'actualSP'
                          "
                          v-model.number="task.actualSP"
                          type="number"
                          @blur="stopEditing"
                          @keyup.enter="stopEditing"
                          class="bg-slate-700 border-slate-600 text-white w-20"
                          autofocus
                        />
                        <span
                          v-else
                          @click="startEditing(task.id, 'actualSP')"
                          class="cursor-pointer hover:bg-slate-700 px-2 py-1 rounded"
                        >
                          {{ task.actualSP }} SP
                        </span>
                      </TableCell>
                      <TableCell>
                        <!-- <Button
                          @click="addTask"
                          size="sm"
                          variant="ghost"
                          class="text-gray-400 hover:text-white"
                        >
                          <Plus class="h-4 w-4" />
                        </Button> -->
                      </TableCell>
                    </TableRow>
                    <!-- Add Table Row -->
                    <TableRow class="border-slate-700 hover:bg-slate-700">
                      <TableCell>
                        <Checkbox class="border-slate-600"
                      /></TableCell>
                      <TableCell>
                        <Button
                          @click="addTask"
                          size="sm"
                          variant="ghost"
                          class="text-gray-400 hover:text-white"
                        >
                          <Plus class="h-4 w-4" /> Add
                        </Button>
                      </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                    </TableRow>
                    <!-- Total Table Row -->
                    <TableRow class="border-slate-700 hover:bg-slate-700">
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> </TableCell>
                      <TableCell>
                        <div>
                          <div class="flex h-2 rounded-full overflow-hidden">
                            <div
                              v-for="(percentage, status) in statusDistribution"
                              :key="status"
                              :style="{ width: percentage + '%' }"
                              :class="getStatusColor(status)"
                              class="h-full"
                            ></div>
                          </div>
                        </div>
                      </TableCell>
                      <TableCell>
                        <div>
                          <div class="flex h-2 rounded-full overflow-hidden">
                            <div
                              v-for="(
                                percentage, priority
                              ) in priorityDistribution"
                              :key="priority"
                              :style="{ width: percentage + '%' }"
                              :class="getPriorityColor(priority)"
                              class="h-full"
                            ></div>
                          </div>
                        </div>
                      </TableCell>
                      <TableCell>
                        <div>
                          <div class="flex h-2 rounded-full overflow-hidden">
                            <div
                              v-for="(percentage, type) in typeDistribution"
                              :key="type"
                              :style="{ width: percentage + '%' }"
                              :class="getTypeColor(type)"
                              class="h-full"
                            ></div>
                          </div>
                        </div>
                      </TableCell>
                      <TableCell> </TableCell>
                      <TableCell> {{ totalEstimatedSP }} SP </TableCell>
                      <TableCell> {{ totalActualSP }} SP </TableCell>
                      <TableCell> </TableCell>
                    </TableRow>
                  </TableBody>
                </Table>
              </div>
            </AccordionContent>
          </AccordionItem>
        </Accordion>
      </div>
      <!-- Kanban -->
      <div v-if="activeTab === 'kanban'" class="text-white">
        <!-- Header Controls -->
        <div class="flex items-center gap-4 mb-6">
          <Button
            @click="openNewTaskModal"
            class="bg-blue-600 hover:bg-blue-700"
          >
            New task
          </Button>
          <div class="relative">
            <Search
              class="absolute left-3 top-1/2 transform -translate-y-1/2 h-4 w-4 text-gray-400"
            />
            <Input
              v-model="searchQuery"
              placeholder="Search tasks..."
              class="pl-10 bg-slate-800 border-slate-700 text-white placeholder-gray-400"
            />
          </div>
          <DropdownMenu>
            <DropdownMenuTrigger as-child>
              <Button
                variant="outline"
                class="border-slate-700 text-gray-400 bg-slate-800 hover:bg-slate-700 focus:ring-2 focus:ring-slate-500"
              >
                <User class="mr-2 h-4 w-4" />
                Person
                <span v-if="selectedDevelopers.length > 0" class="ml-1">
                  ({{ selectedDevelopers.length }})
                </span>
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent
              class="bg-slate-800 border border-slate-700 rounded-md shadow-lg max-h-60 overflow-y-auto w-64"
            >
              <!-- Header dengan tombol clear -->
              <div
                class="flex items-center justify-between p-3 border-b border-slate-700"
              >
                <span class="text-gray-400 text-sm">Filter by Developer</span>
                <Button
                  v-if="selectedDevelopers.length > 0"
                  @click.stop="clearAllDevelopers"
                  variant="ghost"
                  size="sm"
                  class="text-red-400 hover:text-red-300 hover:bg-slate-700 h-7 px-2"
                >
                  Clear all
                  <X class="h-3 w-3 ml-1" />
                </Button>
              </div>

              <!-- List developers -->
              <div
                v-for="dev in uniqueDevelopers"
                :key="dev"
                @click="toggleDeveloper(dev)"
                :class="[
                  'flex items-center px-4 py-2 hover:bg-slate-700 transition-colors cursor-pointer',
                  selectedDevelopers.includes(dev) ? 'bg-slate-600' : '',
                ]"
              >
                <!-- <Checkbox
                  :checked="selectedDevelopers.includes(dev)"
                  class="mr-3 border-slate-600"
                /> -->
                <span class="text-gray-400">{{ dev }}</span>
              </div>

              <!-- Empty state -->
              <div
                v-if="uniqueDevelopers.length === 0"
                class="text-gray-500 px-4 py-3 text-sm text-center"
              >
                No developers available
              </div>
            </DropdownMenuContent>
          </DropdownMenu>
          <DropdownMenu>
            <DropdownMenuTrigger as-child>
              <Button
                variant="outline"
                class="border-slate-700 text-gray-400 bg-slate-800 hover:bg-slate-700 focus:ring-2 focus:ring-slate-500"
              >
                <ArrowUpDown class="mr-2 h-4 w-4" />
                Sort
              </Button>
            </DropdownMenuTrigger>
            <DropdownMenuContent
              class="bg-slate-800 border border-slate-700 rounded-md shadow-lg"
            >
              <DropdownMenuItem
                @click="setSorting('task')"
                class="text-gray-400 hover:bg-slate-700"
              >
                Sort by Task
              </DropdownMenuItem>
              <DropdownMenuItem
                @click="setSorting('priority')"
                class="text-gray-400 hover:bg-slate-700"
              >
                Sort by Priority
              </DropdownMenuItem>
              <DropdownMenuItem
                @click="setSorting('status')"
                class="text-gray-400 hover:bg-slate-700"
              >
                Sort by Status
              </DropdownMenuItem>
            </DropdownMenuContent>
          </DropdownMenu>
          <div class="w-64">
            <div
              class="flex h-10 rounded-md overflow-hidden w-full bg-slate-700"
            >
              <div
                v-for="(percentage, status) in statusDistribution"
                :key="status"
                :style="{ width: percentage + '%' }"
                :class="getStatusColor(status)"
                class="h-full transition-all duration-500"
              ></div>
            </div>
          </div>
        </div>

        <!-- Kanban Board -->
        <div
          class="grid gap-6"
          :style="{
            gridTemplateColumns: `repeat(${statusColumns.length}, minmax(0, 1fr))`,
          }"
        >
          <div
            v-for="status in statusColumns"
            :key="status"
            class="bg-slate-800 rounded-lg border border-slate-700"
            @drop="onDrop($event, status)"
            @dragover.prevent
            @dragenter.prevent
          >
            <div
              :class="getStatusColor(status)"
              class="text-white px-4 py-3 rounded-t-lg flex items-center justify-between"
            >
              <h3 class="font-semibold">{{ status }}</h3>
              <span
                :class="
                  getStatusColor(status)
                    .replace('bg-', 'bg-')
                    .replace('-500', '-600')
                "
                class="px-2 py-1 rounded text-sm"
              >
                {{ getTasksByStatus(status).length }}
              </span>
            </div>
            <div class="p-4 space-y-3 min-h-[400px]">
              <div
                v-for="task in getTasksByStatus(status)"
                :key="task.id"
                draggable="true"
                @dragstart="onDragStart($event, task)"
                class="bg-slate-700 rounded-lg p-4 border border-slate-600 hover:border-slate-500 transition-all duration-200 cursor-grab active:cursor-grabbing hover:shadow-lg"
              >
                <h4 class="text-white font-medium mb-3 text-sm leading-tight">
                  <Input
                    v-if="editingTask === task.id && editingField === 'task'"
                    v-model="task.task"
                    @blur="stopEditing"
                    @keyup.enter="stopEditing"
                    class="bg-slate-700 border-slate-600 text-white"
                    autofocus
                  />
                  <span
                    v-else
                    @click="startEditing(task.id, 'task')"
                    class="cursor-pointer hover:bg-slate-700 px-2 py-1 rounded"
                  >
                    {{ task.task }}
                  </span>
                </h4>

                <!-- Priority and Story Points -->
                <div class="flex flex-wrap items-center gap-1 sm:gap-2 mb-3">
                  <Select
                    v-if="
                      editingTask === task.id && editingField === 'priority'
                    "
                    v-model="task.priority"
                    @update:model-value="stopEditing"
                  >
                    <SelectTrigger
                      class="bg-slate-700 border-slate-600 w-20 sm:w-auto"
                    >
                      <SelectValue />
                    </SelectTrigger>
                    <SelectContent class="bg-slate-800 border-slate-700">
                      <SelectItem
                        v-for="priority in priorityOptions"
                        :key="priority"
                        :value="priority"
                        class="text-gray-400"
                      >
                        {{ priority }}
                      </SelectItem>
                    </SelectContent>
                  </Select>

                  <Badge
                    v-else
                    @click="startEditing(task.id, 'priority')"
                    :class="getPriorityColor(task.priority)"
                    class="cursor-pointer text-xs sm:text-sm px-1.5 sm:px-2 py-0.5 sm:py-1"
                  >
                    {{ task.priority }}
                  </Badge>

                  <span class="text-slate-300 text-xs sm:text-sm">
                    <Input
                      v-if="
                        editingTask === task.id &&
                        editingField === 'estimatedSP'
                      "
                      v-model.number="task.estimatedSP"
                      type="number"
                      @blur="stopEditing"
                      @keyup.enter="stopEditing"
                      class="bg-slate-700 border-slate-600 text-white w-12 sm:w-20 text-xs sm:text-sm"
                      autofocus
                    />
                    <span
                      v-else
                      @click="startEditing(task.id, 'estimatedSP')"
                      class="cursor-pointer hover:bg-slate-700 px-1 sm:px-2 py-0.5 sm:py-1 rounded text-xs sm:text-sm"
                    >
                      {{ task.estimatedSP }} SP
                    </span>
                  </span>

                  <Select
                    v-if="editingTask === task.id && editingField === 'type'"
                    v-model="task.type"
                    @update:model-value="stopEditing"
                  >
                    <SelectTrigger
                      class="bg-slate-700 border-slate-600 w-20 sm:w-auto"
                    >
                      <SelectValue />
                    </SelectTrigger>
                    <SelectContent class="bg-slate-800 border-slate-700">
                      <SelectItem
                        v-for="type in typeOptions"
                        :key="type"
                        :value="type"
                        class="text-gray-400"
                      >
                        {{ type }}
                      </SelectItem>
                    </SelectContent>
                  </Select>

                  <Badge
                    v-else
                    @click="startEditing(task.id, 'type')"
                    :class="getTypeColor(task.type)"
                    class="cursor-pointer text-xs sm:text-sm px-1.5 sm:px-2 py-0.5 sm:py-1 ml-auto sm:ml-0"
                  >
                    {{ task.type }}
                  </Badge>
                </div>

                <!-- Bottom row with avatar and actions -->
                <div class="flex items-center justify-between">
                  <!-- Avatar -->
                  <TooltipProvider>
                    <div class="flex -space-x-2">
                      <!-- 3 Developer pertama -->
                      <div
                        v-for="(dev, index) in task.developer.slice(0, 3)"
                        :key="dev"
                        class="w-8 h-8 rounded-full bg-gradient-to-br from-blue-400 to-purple-500 flex items-center justify-center text-xs font-medium border-2 border-white relative z-10"
                        :style="{ zIndex: task.developer.length - index }"
                      >
                        <Tooltip>
                          <TooltipTrigger as-child>
                            <div
                              class="w-full h-full flex items-center justify-center cursor-pointer"
                            >
                              {{ dev.charAt(0).toUpperCase() }}
                            </div>
                          </TooltipTrigger>
                          <TooltipContent
                            class="bg-slate-800 border-slate-700 text-white"
                          >
                            <p>{{ dev }}</p>
                          </TooltipContent>
                        </Tooltip>
                      </div>

                      <!-- Jika lebih dari 3 developer -->
                      <div
                        v-if="task.developer.length > 3"
                        class="w-8 h-8 rounded-full bg-gray-500 flex items-center justify-center text-xs font-medium text-white border-2 border-white relative"
                        :style="{ zIndex: 0 }"
                      >
                        <Tooltip>
                          <TooltipTrigger as-child>
                            <div
                              class="w-full h-full flex items-center justify-center cursor-pointer"
                            >
                              +{{ task.developer.length - 3 }}
                            </div>
                          </TooltipTrigger>
                          <TooltipContent
                            class="bg-slate-800 border-slate-700 text-white max-w-xs"
                          >
                            <p class="font-medium mb-1">Other developers:</p>
                            <div class="space-y-1">
                              <p
                                v-for="otherDev in task.developer.slice(3)"
                                :key="otherDev"
                                class="text-sm"
                              >
                                {{ otherDev }}
                              </p>
                            </div>
                          </TooltipContent>
                        </Tooltip>
                      </div>
                    </div>
                  </TooltipProvider>

                  <!-- Action icons -->
                  <div class="flex items-center gap-2">
                    <MessageCircle class="w-4 h-4 text-slate-400" />
                    <ListTree class="w-4 h-4 text-slate-400" />
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <!-- Modal Form  -->
  <div
    v-if="showNewTaskModal"
    class="fixed inset-0 bg-black/90 flex items-center justify-center z-50"
  >
    <div
      class="bg-slate-800 rounded-lg p-6 w-full max-w-md mx-4 overflow-y-auto max-h-[90vh]"
    >
      <div class="flex items-center justify-between mb-4">
        <h2 class="text-xl font-semibold text-white">Create New Task</h2>
        <button
          @click="closeNewTaskModal"
          class="text-gray-400 hover:text-white"
        >
          <X class="h-5 w-5" />
        </button>
      </div>

      <form @submit.prevent="submitNewTask" class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Task Name</label
          >
          <Input
            v-model="newTaskForm.task"
            placeholder="Enter task name..."
            class="bg-slate-700 border-slate-600 text-white placeholder-gray-400"
            required
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2">
            Developer
          </label>
          <div
            class="flex flex-wrap gap-2 border border-slate-600 rounded-md p-2 bg-slate-700"
          >
            <!-- Tags -->
            <span
              v-for="(dev, index) in newTaskForm.developer"
              :key="index"
              class="bg-slate-600 text-white px-2 py-1 rounded flex items-center gap-1"
            >
              {{ dev }}
              <button
                type="button"
                @click="removeDeveloper(index)"
                class="text-xs"
              >
                ×
              </button>
            </span>

            <!-- Input baru + tombol tambah -->
            <div class="flex gap-2 flex-1">
              <input
                v-model="newDeveloper"
                placeholder="Type developer"
                class="bg-slate-700 text-white flex-1 outline-none py-1 px-2 rounded"
              />
              <button
                type="button"
                @click="addDeveloper"
                class="bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
              >
                Add
              </button>
            </div>
          </div>

          <!-- Debug: tampilkan array developer -->
          <p class="mt-2 text-sm text-gray-400">
            Current developers: {{ newTaskForm.developer }}
          </p>
        </div>
        <!-- 
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Developer</label
          >
          <select
            v-model="newTaskForm.developer"
            class="w-full bg-slate-700 border border-slate-600 rounded-md px-3 py-2 text-white"
            required
          >
            <option value="">Select Developer</option>
            <option value="John Doe">John Doe</option>
            <option value="Jane Smith">Jane Smith</option>
            <option value="Mike Johnson">Mike Johnson</option>
            <option value="Sarah Wilson">Sarah Wilson</option>
          </select>
        </div> -->

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Status</label
          >
          <select
            v-model="newTaskForm.status"
            class="w-full bg-slate-700 border border-slate-600 rounded-md px-3 py-2 text-white"
            required
          >
            <option value="Ready to Start">Ready To Start</option>
            <option value="In Progress">In Progress</option>
            <option value="Waiting for Review">Waiting for Review</option>
            <option value="Pending Deploy">Pending Deploy</option>
            <option value="Done">Done</option>
            <option value="Stuck">Stuck</option>
          </select>
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Priority</label
          >
          <select
            v-model="newTaskForm.priority"
            class="w-full bg-slate-700 border border-slate-600 rounded-md px-3 py-2 text-white"
            required
          >
            <option value="Best Effort">Best Effort</option>
            <option value="Low">Low</option>
            <option value="Medium">Medium</option>
            <option value="High">High</option>
            <option value="Critical">Critical</option>
          </select>
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Type</label
          >
          <select
            v-model="newTaskForm.type"
            class="w-full bg-slate-700 border border-slate-600 rounded-md px-3 py-2 text-white"
            required
          >
            <option value="Feature Enhancements">Feature Enhancements</option>
            <option value="Bug">Bug</option>
            <option value="Other">Other</option>
          </select>
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Due Date</label
          >
          <Input
            v-model="newTaskForm.date"
            type="date"
            class="bg-slate-700 border-slate-600 text-white"
            required
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Estimated Story Points</label
          >
          <Input
            v-model.number="newTaskForm.estimatedSP"
            type="number"
            min="0"
            placeholder="0"
            class="bg-slate-700 border-slate-600 text-white placeholder-gray-400"
          />
        </div>
        <div>
          <label class="block text-sm font-medium text-gray-300 mb-2"
            >Actual Story Points</label
          >
          <Input
            v-model.number="newTaskForm.actualSP"
            type="number"
            min="0"
            placeholder="0"
            class="bg-slate-700 border-slate-600 text-white placeholder-gray-400"
          />
        </div>

        <div class="flex gap-3 pt-4">
          <Button type="submit" class="flex-1 bg-blue-600 hover:bg-blue-700">
            Create Task
          </Button>
          <Button
            type="button"
            @click="closeNewTaskModal"
            variant="outline"
            class="flex-1 border-slate-600 text-black hover:bg-slate-700"
          >
            Cancel
          </Button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import {
  Home,
  ListTree,
  MessageCircle,
  MessageCirclePlus,
} from "lucide-vue-next";
import {
  Accordion,
  AccordionContent,
  AccordionItem,
  AccordionTrigger,
} from "../../components/ui/accordion";

import { ref, computed, onMounted } from "vue";
import { Search, User, ArrowUpDown, Plus } from "lucide-vue-next";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Checkbox } from "@/components/ui/checkbox";
import { Badge } from "@/components/ui/badge";
import {
  DropdownMenu,
  DropdownMenuContent,
  DropdownMenuTrigger,
  DropdownMenuItem,
} from "@/components/ui/dropdown-menu";
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from "@/components/ui/select";
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from "@/components/ui/table";
import TooltipProvider from "../ui/tooltip/TooltipProvider.vue";
import Tooltip from "../ui/tooltip/Tooltip.vue";
import TooltipTrigger from "../ui/tooltip/TooltipTrigger.vue";
import TooltipContent from "../ui/tooltip/TooltipContent.vue";

interface Task {
  id: string;
  task: string;
  developer: string[];
  status: string;
  priority: string;
  type: string;
  date: string;
  estimatedSP: number;
  actualSP: number;
}

const tasks = ref<Task[]>([]);
const searchQuery = ref("");
const selectedDevelopers = ref<string[]>([]);

const showAllTasks = ref(true);
const editingTask = ref<string | null>(null);
const editingField = ref<string | null>(null);
const sortBy = ref<string>("");
const sortOrder = ref<"asc" | "desc">("asc");

const activeTab = ref("maintable");

const showNewTaskModal = ref(false);

const statusOptions = [
  "Ready to Start",
  "In Progress",
  "Waiting for Review",
  "Pending Deploy",
  "Done",
  "Stuck",
];
const priorityOptions = ["Critical", "High", "Medium", "Low", "Best Effort"];
const typeOptions = ["Feature Enhancements", "Other", "Bug"];

// Fetch data from API
onMounted(async () => {
  try {
    const response = await fetch(
      "https://mocki.io/v1/4e602db4-efab-438f-a664-bec50fc16f7e"
    );
    // const data = await response.json();
    const result = await response.json();

    const data: Task[] = result.data.map((item: Task) => {
      let newStatus = item.status;

      if (newStatus === "Ready to start") {
        newStatus = "Ready to Start";
      } else if (newStatus === "Waiting for review") {
        newStatus = "Waiting for Review";
      }

      return {
        ...item,
        status: newStatus,
      };
    });
    // console.log("Data : ", data);
    tasks.value = data.map((item: any, index: number) => ({
      id: `task-${index}`,
      task: item.title || "New Task",
      // developer: Array.isArray(item.developer)
      //   ? item.developer
      //   : [item.developer || "John Doe"],
      developer: Array.isArray(item.developer)
        ? item.developer
        : item.developer
        ? item.developer.split(",").map((d: any) => d.trim())
        : ["John Doe"],

      status: item.status || "Ready to Start",
      priority: item.priority || "Medium",
      type: item.type || "Other",
      date: item.date || new Date().toISOString().split("T")[0],
      estimatedSP: item["Estimated SP"] || 0,
      actualSP: item["Actual SP"] || 0,
    }));
  } catch (error) {
    console.error("Failed to fetch tasks:", error);
    // Fallback data if error
    tasks.value = [
      {
        id: "1",
        task: "New Task",
        developer: ["John"],
        status: "Ready to Start",
        priority: "Medium",
        type: "Feature Enhancements",
        date: "2024-01-15",
        estimatedSP: 2,
        actualSP: 0,
      },
    ];
  }
});

const uniqueDevelopers = computed(() => {
  const devs = new Set<string>();
  tasks.value.forEach((task) => {
    task.developer.forEach((dev) => devs.add(dev));
  });
  return Array.from(devs);
});

const filteredTasks = computed(() => {
  let filtered = tasks.value;

  // console.log("Filtering - selected developers:", selectedDevelopers.value);
  // console.log("Filtering - search query:", searchQuery.value);

  // Search filter
  if (searchQuery.value) {
    filtered = filtered.filter((task) =>
      task.task.toLowerCase().includes(searchQuery.value.toLowerCase())
    );
  }

  // Developer filter
  if (selectedDevelopers.value.length > 0) {
    filtered = filtered.filter((task) =>
      task.developer.some((dev) => selectedDevelopers.value.includes(dev))
    );
  }

  // Sorting
  if (sortBy.value) {
    filtered = [...filtered].sort((a, b) => {
      const aVal = a[sortBy.value as keyof Task];
      const bVal = b[sortBy.value as keyof Task];

      if (sortOrder.value === "asc") {
        return aVal > bVal ? 1 : -1;
      } else {
        return aVal < bVal ? 1 : -1;
      }
    });
  }

  return filtered;
});

// const allSelected = computed(() => {
//   return (
//     tasks.value.length > 0 && selectedTasks.value.length === tasks.value.length
//   );
// });

// Distribution calculations
const statusDistribution = computed(() => {
  const total = tasks.value.length;
  const counts: Record<string, number> = {};

  statusOptions.forEach((status) => {
    counts[status] = tasks.value.filter(
      (task) => task.status === status
    ).length;
  });

  const percentages: Record<string, number> = {};
  Object.keys(counts).forEach((status) => {
    percentages[status] = total > 0 ? (counts[status] / total) * 100 : 0;
  });

  return percentages;
});

const priorityDistribution = computed(() => {
  const total = tasks.value.length;
  const counts: Record<string, number> = {};

  priorityOptions.forEach((priority) => {
    counts[priority] = tasks.value.filter(
      (task) => task.priority === priority
    ).length;
  });

  const percentages: Record<string, number> = {};
  Object.keys(counts).forEach((priority) => {
    percentages[priority] = total > 0 ? (counts[priority] / total) * 100 : 0;
  });

  return percentages;
});

const typeDistribution = computed(() => {
  const total = tasks.value.length;
  const counts: Record<string, number> = {};

  typeOptions.forEach((type) => {
    counts[type] = tasks.value.filter((task) => task.type === type).length;
  });

  const percentages: Record<string, number> = {};
  Object.keys(counts).forEach((type) => {
    percentages[type] = total > 0 ? (counts[type] / total) * 100 : 0;
  });

  return percentages;
});

const totalEstimatedSP = computed(() => {
  return tasks.value.reduce((sum, task) => sum + task.estimatedSP, 0);
});

const totalActualSP = computed(() => {
  return tasks.value.reduce((sum, task) => sum + task.actualSP, 0);
});

// Color functions
const getStatusColor = (status: string) => {
  const colors: Record<string, string> = {
    "Ready to Start": "bg-blue-500",
    "In Progress": "bg-yellow-500",
    "Waiting for Review": "bg-orange-500",
    "Pending Deploy": "bg-purple-500",
    Done: "bg-green-500",
    Stuck: "bg-red-500",
  };
  return colors[status] || "bg-gray-500";
};

const getPriorityColor = (priority: string) => {
  const colors: Record<string, string> = {
    Critical: "bg-red-600",
    High: "bg-orange-500",
    Medium: "bg-yellow-500",
    Low: "bg-blue-500",
    "Best Effort": "bg-slate-400",
  };
  return colors[priority] || "bg-gray-500";
};

const getTypeColor = (type: string) => {
  const colors: Record<string, string> = {
    "Feature Enhancements": "bg-green-500",
    Other: "bg-purple-500",
    Bug: "bg-red-500",
  };
  return colors[type] || "bg-gray-500";
};

// Event handlers
const addNewTask = () => {
  const newTask: Task = {
    id: `task-${Date.now()}`,
    task: "New Task",
    developer: ["John Doe"],
    status: "Ready to Start",
    priority: "Medium",
    type: "Other",
    date: new Date().toISOString().split("T")[0],
    estimatedSP: 0,
    actualSP: 0,
  };
  tasks.value.unshift(newTask);
};

const addTask = () => {
  addNewTask();
};

const toggleDeveloper = (dev: string) => {
  const currentDevs = [...selectedDevelopers.value];
  const index = currentDevs.indexOf(dev);

  if (index > -1) {
    currentDevs.splice(index, 1);
  } else {
    currentDevs.push(dev);
  }

  selectedDevelopers.value = currentDevs;
  // console.log("Selected developers:", selectedDevelopers.value);
};

// const removeDeveloper = (dev: string) => {
//   selectedDevelopers.value = selectedDevelopers.value.filter((d) => d !== dev);
// };

const clearAllDevelopers = () => {
  selectedDevelopers.value = [];
};

const selectedTasks = ref<string[]>([]);

// Computed properties untuk select all
const allSelected = computed((): boolean => {
  return (
    filteredTasks.value.length > 0 &&
    selectedTasks.value.length === filteredTasks.value.length
  );
});

const allIndeterminate = computed((): boolean => {
  return (
    selectedTasks.value.length > 0 &&
    selectedTasks.value.length < filteredTasks.value.length
  );
});

// Function untuk toggle all selection - YANG BENAR
const toggleAllSelection = (checked: boolean | "indeterminate"): void => {
  console.log("Toggle all called with:", checked);
  console.log("Filtered tasks:", filteredTasks.value.length);
  console.log("Currently selected:", selectedTasks.value.length);
  if (checked === true) {
    // Select semua task yang terfilter
    selectedTasks.value = filteredTasks.value.map((task) => task.id);
  } else {
    // Clear semua selection
    selectedTasks.value = [];
  }
};

// Function untuk toggle individual task
const toggleTaskSelection = (taskId: string, checked: boolean): void => {
  if (checked) {
    if (!selectedTasks.value.includes(taskId)) {
      selectedTasks.value.push(taskId);
    }
  } else {
    selectedTasks.value = selectedTasks.value.filter((id) => id !== taskId);
  }
};

const setSorting = (field: string) => {
  if (sortBy.value === field) {
    sortOrder.value = sortOrder.value === "asc" ? "desc" : "asc";
  } else {
    sortBy.value = field;
    sortOrder.value = "asc";
  }
};

const startEditing = (taskId: string, field: string) => {
  editingTask.value = taskId;
  editingField.value = field;
};

const stopEditing = () => {
  editingTask.value = null;
  editingField.value = null;
};

const formatDate = (dateString: string) => {
  const date = new Date(dateString);
  return date.toLocaleDateString("en-US", {
    day: "2-digit",
    month: "short",
    year: "numeric",
  });
};

const getTasksByStatus = (status: string) => {
  return filteredTasks.value.filter((task) => task.status === status);
};

const statusColumns = computed(() => {
  const colors: Record<string, string> = {
    "Ready to Start": "bg-blue-500",
    "In Progress": "bg-yellow-500",
    "Waiting for Review": "bg-orange-500",
    "Pending Deploy": "bg-purple-500",
    Done: "bg-green-500",
    Stuck: "bg-red-500",
  };
  return Object.keys(colors);
});

const onDragStart = (event: DragEvent, task: Task) => {
  if (event.dataTransfer) {
    event.dataTransfer.setData("text/plain", task.id);
    event.dataTransfer.effectAllowed = "move";
  }
};

const onDrop = (event: DragEvent, newStatus: string) => {
  event.preventDefault();
  if (event.dataTransfer) {
    const taskId = event.dataTransfer.getData("text/plain");
    const taskIndex = tasks.value.findIndex((task) => task.id === taskId);

    if (taskIndex !== -1) {
      tasks.value[taskIndex].status = newStatus;
      // Save to localStorage
      localStorage.setItem("tasks", JSON.stringify(tasks.value));
    }
  }
};

// New task form
const newTaskForm = ref<Task>({
  id: `task-${Date.now()}`,
  task: "New Task",
  developer: [],
  status: "Ready to Start",
  priority: "Medium",
  type: "Other",
  date: new Date().toISOString().split("T")[0],
  estimatedSP: 0,
  actualSP: 0,
});

const openNewTaskModal = () => {
  showNewTaskModal.value = true;
};

const closeNewTaskModal = () => {
  showNewTaskModal.value = false;
  // Reset form
  newTaskForm.value = {
    id: `task-${Date.now()}`,
    task: "New Task",
    developer: [],
    status: "Ready to Start",
    priority: "Medium",
    type: "Other",
    date: new Date().toISOString().split("T")[0],
    estimatedSP: 0,
    actualSP: 0,
  };
};

const submitNewTask = () => {
  const newTask = {
    id: `task-${Date.now()}`,
    task: newTaskForm.value.task,
    developer: newTaskForm.value.developer,
    status: newTaskForm.value.status,
    priority: newTaskForm.value.priority,
    type: newTaskForm.value.type,
    date: newTaskForm.value.date,
    estimatedSP: newTaskForm.value.estimatedSP,
    actualSP: newTaskForm.value.actualSP,
  };

  tasks.value.unshift(newTask);
  closeNewTaskModal();
};

const newDeveloper = ref<string>("");
// Tambah developer
function addDeveloper() {
  const dev: string = newDeveloper.value.trim();
  if (dev && !newTaskForm.value.developer.includes(dev)) {
    newTaskForm.value.developer.push(dev);
  }
  newDeveloper.value = "";
}

// Hapus developer
function removeDeveloper(index: number) {
  newTaskForm.value.developer.splice(index, 1);
}
</script>

<style scoped>
/*  styles  */
</style>
